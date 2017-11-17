# Order fulfillment sample application demonstrating concepts in the context of DDD and Microservices. 

This sample application shows how to implement

* a simple order fulfillment system

in the context of

* Domain Driven Design (DDD)
* Event Driven Architecture (EDA)
* Microservices (µS)

## Links

* Introduction blog post by Bernd Rücker: https://blog.bernd-ruecker.com/flowing-retail-demonstrating-aspects-of-microservices-events-and-their-flow-with-concrete-source-7f3abdd40e53
* Slides from a talk at MuCon London: https://www.slideshare.net/BerndRuecker/mucon-london-2017-break-your-event-chains, recording available: https://skillsmatter.com/skillscasts/10718-break-your-event-chains
* Slides from a talk at KanDDDinsky Berlin: https://www.slideshare.net/BerndRuecker/kandddinsky-let-your-domain-events-flow, recording available: https://www.youtube.com/watch?v=LKoaMbqZp9Y


# Overview

Flowing retail simulates a very easy order fulfillment system. The business logic is separated into the following services (shown as [context map](https://www.infoq.com/articles/ddd-contextmapping)):

![Microservices](docs/context-map.png)

## Concrete technologies/frameworks:

* Java
* Spring Boot
* Spring Cloud Streams
* Camunda
* Apache Kafka

## Architecture 

This results in the following architecture:

![Microservices](docs/architecture.png)

## Communication of services

The services have to collaborate in order to implement the overall business capability of order fulfillment. There are many possibilities to communicate, this example focues on:

* *Asynchronous* communication via Apache Kafka 
* *Event-driven* wherever appropriate
* Sending *Commands* in cases you want somebody to do something, which involves that events need to be transformed into events from the component responsible for, which in our case is the Order service:

![Events and Commands](docs/event-command-transformation.png)

## Potentially long running services and distributed orchestration

Typically long running services allow for a better service API. For example Payment might clear problems with the credit card handling itself, which could even involve to ask the customer to add a new credit card in case his is expired. So the service might have to wait for days or weeks, making it long running. This requires to handle state, that's why a state machine like Camunda is used.

An important thought is, that this state machine (or workflow engine in this case) is a library used *within* one service. It runs embedded within the Spring Boot application, and if different services need this, they run engines on their own. It is an autonomous team decision if they want to use some framework and which one:

![Events and Commands](docs/workflow-in-service.png)


# Run the application

* Download or clone the source code
* Run a full maven build

```
mvn install
```

* Install and start Kafka on the standard port
* Create topic *"flowing-retail"*

```
kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic flowing-retail
```

* You can check & query all topics by: 

```
kafka-topics.sh --list --zookeeper localhost:2181
```

* Start the different microservices components by Spring Boot one by one, e.g.
    
```
mvn -f checkout exec:java
...
```

You can also import the projects into your favorite IDE and start the following class yourself:

```
checkout/io.flowing.retail.java.CheckoutApplication
...
```

* Now you can place an order via [http://localhost:8090](http://localhost:8090)
* You can inspect insided of Order via [http://localhost:8091](http://localhost:8091)
* You can inspect insides of Payment via [http://localhost:8092](http://localhost:8092)
* You can inspect all events going on via [http://localhost:8095](http://localhost:8095)

