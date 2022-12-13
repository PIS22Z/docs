## New Kotlin proposal - context receivers

One of the more special features loved by Kotlin developers migrating from the Java environment is Kotlin's extension functions. They allow to extend of the functionality of existing classes without having to inherit from them or use design patterns such as decorators. Without a doubt, we can say that they allow us to write cleaner, more readable code. However, they don't come without limitations. One of them is using them nested inside our classes. Look at the following code: 

```kotlin
class Foo

class Bar { // Bar class is a dispatch receiver
    private fun innerFun() = Unit
    fun Foo.toBar() { // Foo class is an extension receiver for toBar()
        innerFun()
    }
}
```

Here we see that an extension function is defined in class Bar, which, based on the place of definition, we want to use only in the context of the Bar class. It may be beneficial if we want the function to access and use Bar class attributes and methods. Although it is easy and fun to use it inside the declaration of the class, using it outside forces us to have the Bar class context. Such construction with the help of Kotlin's scope functions (`apply()`, `with()`, `run`) may look like this:

```kotlin
val bar = Bar()
with(bar) {
    Foo().toBar()
}
```

### Limitations

This great feature allows the creation of context-dependent functions and the representation of the context of an action using a dispatch receiver. However, member extensions have limitations that can restrict their usefulness. Using them in the context of third-party classes may be hard due to their modularity. Also, another drawback is their syntax. For example, we would like to have a function `sendReceipt()` which may only be used in  TransactionScope but forbidden to be used as `transaction.sendReceipt()` since we want them to not work **on** a transaction, but **in the context of** a transaction. Another limitation can be reached when needing multiple contexts (let's say not only `TransactionScope` but also `CorutineScope`) - we cannot declare a function that must be called only within two or more scopes present at the same time.

### Context receivers to the rescue?

As of Kotlin version 1.6.20, there has appeared a new experimental feature called **context receivers** which overcomes highlighted limitations and covers a variety of use cases. They introduce a new syntax for defining context-dependent declarations that makes use of special context receivers. This new syntax addresses the limitations of current methods and can be applied to various use cases. The context in this context is not directly tied to the action itself but rather is used by the action to provide additional operations, configuration, or execution context. So our TransactionScope example can be improved to use a top-level function in a way:

```kotlin
context(Transaction)
fun sendReceipt() {
    // ...
}
```

Its key difference from the `Transaction.sendReceipt()` extension is that it cannot be called with a qualified `transaction.doAction()` syntax, and it has no this reference inside its body since it has no object on which it performs its action. Moreover, there can be multiple context receivers by just supplying them separated by a comma. 

```kotlin
interface LoggingContext {
    val logger: Logger
        get() = Logger.getLogger(this.javaClass.name)
}

context(Transaction, Logger)
fun sendReceipt() {
    // ...
}
```

Another example may be useful in the Android ecosystem when calculating a size of a view in form of density independent pixels:

```kotlin
context(View)
val Float.dp get() = this * resources.displayMetrics.density

context(View)
val Int.dp get() = this.toFloat().dp

// usage
with(view) {
    val size = measure().dp
}
```

Here we can see that both functions can be used only in the context of a View which provides a resources member to retrieve the phone pixel density. Also, here, we want to use this function not on views themselves but in their context. 

### What is the future? 

Although this feature looks like an extraordinary opportunity at first glance, it's still marked as experimental (and for a good reason!). There are still some concerns about it  e.g., abusing scopes and context receivers' scopes:

```kotlin
class Example {
    fun Scope.doAction() {
        builder {
        	anotherBuider {
                exampleContextReceiver() // wherer this function is declared and which one to use?
            }    
        }
    }
}
```

 or resolution ambiguity: 

```kotlin
context(LoggingContext)
fun weirdToString(): String = toString() // OK

fun LoggingContext.weirdToString(): String = toString() // also OK

context(LoggingContext, Transaction)
fun weirdToString(): String = toString() // ERROR should it use Transaction.toString() or LogginContext.toString()?
```



So it's hard to tell what changes will still be made, but it looks very promising for complex developers solutions. And keeping in mind its experimental state, you can freely use with. All you need to do is to add `-Xcontext-receivers` to compiler args in your `build.gradle` file and provided you use a newer Kotlin version than 1.6.20, you can play with contexts receivers on your own!

Happy Coding and see you in the next post 

Wojciech Kołodziejak