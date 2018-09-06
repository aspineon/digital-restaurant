['d-restaurant'](https://github.com/idugalic/digital-restaurant) is an example of an application that is built using Event Sourcing and CQRS. The application is written in Kotlin, and uses Spring Boot. It is built using Axonframework, which is an application framework based on event sourcing and CQRS.

Customers use the website application to place food orders at local restaurants. Application coordinates a network of couriers who deliver the orders.

### Microservices

We designed and structured our [loosely coupled components](https://github.com/idugalic/digital-restaurant/tree/master/drestaurant-libs) in a modular way, 
and that enables us to choose different deployment strategy and take first step towards Microservices architectural style.

Each [microservice](https://github.com/idugalic/digital-restaurant/tree/master/drestaurant-apps/drestaurant-microservices):

 - has its own bounded context,
 - has a specific JPA event store (we are not sharing the JPA Event Store)
 - and we distribute events between them via Apache Kafka
 
#### 'Command' HTTP API

##### Create new Restaurant
```
curl -X POST --header 'Content-Type: application/json' --header 'Accept: */*' -d '{
  "menuItems": [
    {
      "id": "id1",
      "name": "name1",
      "price": 100
    }
  ],
  "name": "Fancy"
}' 'http://localhost:8084/api/command/restaurant/createcommand'
```
##### Create/Register new Customer
```
curl -X POST --header 'Content-Type: application/json' --header 'Accept: */*' -d '{
  "firstName": "Ivan",
  "lastName": "Dugalic",
  "orderLimit": 1000
}' 'http://localhost:8082/api/command/customer/createcommand'
```
##### Create/Hire new Courier
```
curl -X POST --header 'Content-Type: application/json' --header 'Accept: */*' -d '{
  "firstName": "John",
  "lastName": "Doe",
  "maxNumberOfActiveOrders": 20
}' 'http://localhost:8081/api/command/courier/createcommand'
```
##### Create/Place the Order
```
curl -X POST --header 'Content-Type: application/json' --header 'Accept: */*' -d '{
  "customerId": "CUSTOMER_ID",
  "orderItems": [
    {
      "id": "id1",
      "name": "name1",
      "price": 100,
      "quantity": 0
    }
  ],
  "restaurantId": "RESTAURANT_ID"
}' 'http://localhost:8083/api/command/order/createcommand'
```
Note: Replace CUSTOMER_ID and RESTAURANT_ID with concrete values.

##### Restaurant marks the Order as prepared
```
curl -X POST --header 'Content-Type: application/json' --header 'Accept: */*' 'http://localhost:8084/api/command/restaurant/order/RESTAURANT_ORDER_ID/markpreparedcommand'

```
Note: Replace RESTAURANT_ORDER_ID with concrete value.

##### Courier takes/claims the Order that is ready for delivery (prepared)
```
curl -X POST --header 'Content-Type: application/json' --header 'Accept: */*' 'http://localhost:8081/api/command/courier/COURIER_ID/order/COURIER_ORDER_ID/assigncommand'

```
Note: Replace COURIER_ID and COURIER_ORDER_ID with concrete values.

##### Courier marks the Order as delivered
```
curl -X POST --header 'Content-Type: application/json' --header 'Accept: */*' 'http://localhost:8081/api/command/courier/order/COURIER_ORDER_ID/markdeliveredcommand'

```

#### 'Query' HTTP API
Application is using an event handlers to subscribe to interested domain events. Events are materialized in SQL database schema and distributed over Apache Kafka

HTTP/REST API for browsing the materialized data:

```
curl http://localhost:8085/api/query
```


## Development

This project is driven using [Maven][mvn].

### Clone

```bash
$ git clone https://github.com/idugalic/digital-restaurant
```

### Build

```bash
$ cd digital-restaurant
$ mvn clean install
```


### Run microservices

NOTE: Docker is required. We use it to start Apache Kafka with Zookeeper

```bash
$ cd digital-restaurant/drestaurant-apps/drestaurant-microservices
$ docker-compose up -d
$ cd digital-restaurant/drestaurant-apps/drestaurant-microservices/drestaurant-microservices-command-courier
$ mvn spring-boot:run
$ cd digital-restaurant/drestaurant-apps/drestaurant-microservices/drestaurant-microservices-command-customer
$ mvn spring-boot:run
$ cd digital-restaurant/drestaurant-apps/drestaurant-microservices/drestaurant-microservices-command-restaurant
$ mvn spring-boot:run
$ cd digital-restaurant/drestaurant-apps/drestaurant-microservices/drestaurant-microservices-command-order
$ mvn spring-boot:run
$ cd digital-restaurant/drestaurant-apps/drestaurant-microservices/drestaurant-microservices-query
$ mvn spring-boot:run
```


## Continuous delivery

We have one deployment pipeline for all applications and libraries within this repository. In addition, all projects in the repository share the same dependencies. Hence, there are no version conflicts because everyone has to use the same/the latest (SNAPSHOTS) version. And you don't need to deal with a private NPM (JavaScript) or Maven (Java) registry when you just want to use your own libraries.
This setup and project structure is usually addressed as a [monorepo](https://medium.com/@maoberlehner/monorepos-in-the-wild-33c6eb246cb9).

## Technology

### Language
- [Kotlin][kotlin]

### Frameworks and Platforms
- [Spring (spring boot, spring cloud, spring data, spring data rest)][spring]
- [Axonframework (eventsourcing, CQRS)][axonframework]

### Continuous Integration and Delivery 
- Travis

### Infrastructure and Platform (As A Service)
- [H2, MySQL (event store, materialized views)][mysql]
- [Apache Kafka][kafka]

## References and further reading
**Inspired by the book "Microservices Patterns" - Chris Richardson**

  - https://github.com/microservice-patterns/ftgo-application
  - https://spring.io/blog/2018/04/11/event-storming-and-spring-with-a-splash-of-ddd
  - http://www.codingthearchitecture.com/2016/04/25/layers_hexagons_features_and_components.html
  
 

[mvn]: https://maven.apache.org/
[kotlin]: https://kotlinlang.org/
[spring]: https://spring.io/
[axonframework]: https://axoniq.io/
[mysql]: https://www.mysql.com/
[rabbitMQ]: https://www.rabbitmq.com/
[kafka]: https://kafka.apache.org/
[pivotalCF]: https://run.pivotal.io/