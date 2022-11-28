# PIS 22Z - Etap 2

## Lista user stories (podzielona na iteracje)

### Iteracja 1 (do 11 grudnia)

- Jako administrator, mogę zdefiniować restauracje i ich produkty
- Jako zamawiający, mogę rozpocząć zamówienie i dodawać do niego produkty
- Jako zamawiający mogę sfinalizować zamówienie, przechodząc do podsumowania
- Jako pracownik restauracji, mogę zaakceptować albo odrzucić opłacone zamówienie
- Jako zamawiający mogę opłacić sfinalizowane zamówienie

### Iteracja 2 (do 1 stycznia)

- Jako pracownik restauracji, mogę oznaczyć zamówienie jako gotowe do wydania
- Jako dostawca, mogę zaakceptować zamówienie do dostarczenia
- Jako dostawca, mogę dostarczyć zamówienie kończąc dostawę
- Jako dostawca, mogę odebrać zamówienie rozpoczynając dostawę
- Jako dostawca, powinienem otrzymać zapłatę za dostarczone zamówienie

## Wybrany stos technologiczny

### Środowisko

- IDE: IntelliJ IDEA + Visual Studio Code
- Hosting repozytorium: GitHub
- CI/CD: GitHub Actions
- Repozytorium artefaktów: GitHub Actions Artifacts
- Narzędzie do śledzenia zadań: Linear
- Statyczna analiza kodu: Detekt (lokalnie) + Sonar (cloud)
- Lokalne wdrożenie: docker-compose
- Wdrożenie "produkcyjne": Kubernetes

### Backend

- Język: Kotlin,
- Framework: Micronaut
- Biblioteka do testów: Spock
- Baza danych: MongoDB
- Broker wiadomości: RabbitMQ

### Frontend

- Język: TypeScript
- Framework: React.JS
- Biblioteka do testowania: Jest (unit) + Cypress (e2e)
- Biblioteka do zarządzania stanem: Recoil/Redux
- Biblioteki do stylowania: Styled-components + UI component lib (MUI/Antd)
- Integracja z API: React-query (TanStack)

## Opis architektury aplikacji

3 warstwy:
- frontend w postaci aplikacji webowej z frameworkiem React.js
- backend aplikacja w frameworku Spring Boot
- dokumentowa baza danych MongoDB

Backend podzielony jest również na 4 bounded contexty (reprezentowane w kodzie jako pakiety), niezależne od siebie, z minimalnymi interakcjami pomiędzy sobą. Komunikacja między nimi odbywa się głównie przez brokera wiadomości - RabbitMQ. Całość wytwarzana jest zgodnie z koncepcją architektury heksagonalnej.

Docelowo będą to 4 osobne mikroserwisy.

Całość wdrożona jest na klastrze Kubernetes, jako trzy osobne Deploymenty.

![PIS 1.drawio.png](./PIS%201.drawio.png)

## Demo zrębu projektu

[https://pis.rasztabiga.me/](https://pis.rasztabiga.me/)

## Raport z Linear

TODO

## Materiały

- [https://guides.micronaut.io/latest/index.html](https://guides.micronaut.io/latest/index.html)
- [https://reactjs.org/docs/getting-started.html](https://reactjs.org/docs/getting-started.html)

