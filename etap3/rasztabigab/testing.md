# Integration testing in Micronaut

###### Author: Bartłomiej Rasztabiga

Micronaut is a modern, JVM-based, microservices-oriented framework that helps developers build
efficient and scalable applications. A key aspect of any successful application is thorough testing,
and Micronaut provides several options for testing your application at different levels.

In this post, we'll focus on testing on the integration level, using Spock library for assertions
and mocking. Micronaut's test resources library provides support for running external dependencies
in-place, such as databases or message brokers, allowing you to test your code in real environment.

In the following examples, we'll use Kotlin as the programming language, but the same principles
apply to Java as well.

We'll also be running MongoDB and RabbitMQ as external dependencies. These will be started
automatically by Micronaut's test resources library.

## Setting up test dependencies

First, we need to add the following to our build.gradle.kts file:

```groovy
plugins {
    id("io.micronaut.test-resources")
    id("groovy")
}

micronaut {
    testRuntime("spock2")
    testResources {
        sharedServer.set(true)
    }
}
```

Micronaut will figure out and automatically add any required gradle dependencies.

## Testing application layer service with Spock

Let's start with a simple example of a service that saves an order entity to a database. The service is
defined as follows:

```kotlin
interface CreateOrderHandler {
    fun create(command: CreateOrderCommand): UUID
}

@Singleton
class CreateOrderHandlerImpl(
    private val draftOrderRepository: DraftOrderRepository
) : CreateOrderHandler {
    override fun create(command: CreateOrderCommand): UUID {
        val draftOrder = DraftOrder.new(
            restaurantId = command.restaurantId,
            userId = command.userId,
            items = command.items
        )
        draftOrderRepository.add(draftOrder)
        return draftOrder.id
    }
}

data class CreateOrderCommand(
    val restaurantId: UUID,
    val userId: UUID,
    val items: List<OrderItem>
)
```

The service is implemented as a singleton bean, and it uses an external repository dependency to
save the entity to the database. This repository is using real MongoDB instance, started
automatically by test-resources. We won't go into details when it comes to repository's definition,
but here's its interface:

```kotlin
interface DraftOrderRepository {

    fun add(draftOrder: DraftOrder)

    fun save(draftOrder: DraftOrder)

    fun load(id: UUID): DraftOrder?
}
```

Let's now write a full integration test for this service:

```groovy
@MicronautTest
class CreateOrderHandlerTest extends Specification {

    @Inject
    private CreateOrderHandler handler

    @Inject
    private DraftOrderRepository orderRepository

    def "given nothing, when valid order is created, then it should be saved in db"() {
        given:
        def command = new CreateOrderCommand(UUID.randomUUID(), UUID.randomUUID(),
                [new OrderItem(UUID.randomUUID(), 1)])

        when:
        def id = handler.create(command)

        then:
        def savedOrder = orderRepository.load(id)
        savedOrder != null
        savedOrder.id == id
        savedOrder.userId == command.userId
        !savedOrder.isFinalized()
    }
}
```

The test is using Micronaut's `@MicronautTest` annotation, which starts the application context and
injects all the beans. The test is using Spock's `given`, `when` and `then` blocks to structure the
test. The `given` block is used to prepare the test data, the `when` block is used to execute the
code under test, and the `then` block is used to assert the results.

## Mocking external dependencies

In the previous example, we used a real repository implementation to test the service. This is a
good approach when we want to test the service in real environment, but sometimes we want to mock
the dependencies to isolate the service and test it in isolation. Let's see how we can do that.

Let's extend the previous example to let it send a message to a message broker once an order is
created. The service would be defined as follows:

```kotlin
@Singleton
class CreateOrderHandlerImpl(
    private val draftOrderRepository: DraftOrderRepository,
    private val publisher: DomainEventPublisher
) : CreateOrderHandler {
    override fun create(command: CreateOrderCommand): UUID {
        val draftOrder = DraftOrder.new(
            restaurantId = command.restaurantId,
            userId = command.userId,
            items = command.items
        )
        draftOrderRepository.add(draftOrder)
        publisher.publish(
            OrderFinalizedEvent(
                orderId = draftOrder.id,
                restaurantId = draftOrder.restaurantId,
                userId = draftOrder.userId,
                items = draftOrder.items
            )
        )
        return draftOrder.id
    }
}
```

The service is now using a `DomainEventPublisher` dependency to publish a message to a message
broker once the order is created (finalized). Let's also assume, that we don't want to spin up a
real message broker in our tests, but we want to mock it instead. We can do that by using
Micronaut's `@MockBean` annotation:

```groovy
@MicronautTest
class CreateOrderHandlerTest extends Specification {

    @Inject
    private CreateOrderHandler handler

    @Inject
    private DraftOrderRepository orderRepository

    @Inject
    DomainEventPublisher domainEventPublisher

    def "given nothing, when valid order is created, then it should be saved in db"() {
        given:
        def command = new CreateOrderCommand(UUID.randomUUID(), UUID.randomUUID(),
                [new OrderItem(UUID.randomUUID(), 1)])

        when:
        def id = handler.create(command)

        then:
        def savedOrder = orderRepository.load(id)
        savedOrder != null
        savedOrder.id == id
        savedOrder.userId == command.userId
        !savedOrder.isFinalized()
    }

    @MockBean(DomainEventPublisher)
    DomainEventPublisher domainEventPublisher() {
        Mock(DomainEventPublisher)
    }
}
```

We've added a new field to the test class, which is an instance of the `DomainEventPublisher`
interface. This field is injected by Micronaut's test context. Remember, that mocked field has to have public visibility! Otherwise, Micronaut won't be able to inject it to other beans.

We've also added a new method to the test class, which is annotated with `@MockBean` annotation.
This annotation tells Micronaut to replace the bean of type `DomainEventPublisher` with a mock
implementation.

Now we can mock interactions with the message broker in our tests:

```groovy
@MicronautTest
class CreateOrderHandlerTest extends Specification {

    @Inject
    private CreateOrderHandler handler

    @Inject
    private DraftOrderRepository orderRepository

    @Inject
    DomainEventPublisher domainEventPublisher

    def "given nothing, when valid order is created, then it should be saved in db and an event published"() {
        given:
        def command = new CreateOrderCommand(UUID.randomUUID(), UUID.randomUUID(),
                [new OrderItem(UUID.randomUUID(), 1)])

        when:
        def id = handler.create(command)

        then:
        def savedOrder = orderRepository.load(id)
        savedOrder != null
        savedOrder.id == id
        savedOrder.userId == command.userId
        !savedOrder.isFinalized()

        1 * domainEventPublisher.publish(_ as OrderFinalizedEvent)
    }

    @MockBean(DomainEventPublisher)
    DomainEventPublisher domainEventPublisher() {
        Mock(DomainEventPublisher)
    }
}
```

We've added a new test case to the test class, which is testing the service's interaction with the
message broker. The test is using Spock's `1 *` syntax to verify that the `publish` method was
called exactly once. The `1 *` syntax is a shorthand for `1 * _ * _`, which means that the method
was called exactly once, with any arguments. We can also verify that the method was called with a
specific argument, for example:

```groovy
1 * domainEventPublisher.publish(new OrderFinalizedEvent(
        orderId = id,
        restaurantId = command.restaurantId,
        userId = command.userId,
        items = command.items
))
```

## Resources for learning more

* [Micronaut testing documentation](https://docs.micronaut.io/latest/guide/index.html#test)
* [Micronaut test library documentation](https://micronaut-projects.github.io/micronaut-test/latest/guide/)
* [Spock documentation](http://spockframework.org/spock/docs/2.3/index.html)
