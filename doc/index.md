# fiware-cosmos-orion-flink-connector

[![](https://nexus.lab.fiware.org/static/badges/chapters/processing.svg )](https://www.fiware.org/developers/catalogue/)
[![License](https://img.shields.io/github/license/ging/fiware-cosmos-orion-flink-connector.svg)](#)
[![](https://img.shields.io/badge/tag-fiware--cosmos-orange.svg?logo=stackoverflow)](http://stackoverflow.com/questions/tagged/fiware-cosmos)
<br/>
[![Documentation badge](https://readthedocs.org/projects/fiware-cosmos-flink/badge/?version=latest)](http://fiware-cosmos-flink.rtfd.io)
[![Status](https://nexus.lab.fiware.org/static/badges/statuses/cosmos.svg)](#)


The Cosmos Generic Enabler enables an easier BigData analysis over context integrated with some of the most popular BigData platforms.

### WHAT IS COSMOS?

The Cosmos BigData Analysis GE is a set of tools that help achieving the tasks
of Streaming and Batch processing over context data. These tools are:

* Orion-Flink Connector (Source and Sink)
* Apache Flink Processing Engine
* Apache Spark Processing Engine (work in progress)
* Streaming processing examples using Orion Context Broker

### WHY USE COSMOS?

As the state of the real world changes, the entities representing your
IoT devices are constantly changing. Big data analysis allows for the
study of datasets coming from your context data which are too large
for traditional data-processing software. You can apply predictive
analysis or user behaviour analytics to extract meaningful conclusions
as to the state of your smart solution and bring value to your
solution. 

### Orion Flink Connector

This is a Flink connector for the Fiware Orion Context Broker.
It has two parts:

 * **`OrionSource`**: Source for receiving NGSIv2 events in the shape of HTTP messages from subscriptions.
 
 * **`OrionSink`**: Sink for writing back to the Context Broker.

Several examples are provided to facilitate getting started with the connector. They are hosted in a separate repository: 
[fiware-cosmos-orion-flink-connector-examples](https://github.com/ging/fiware-cosmos-orion-flink-connector-examples).

## Installation

Download the JAR from the latest release.
In your project directory run:
```bash
mvn install:install-file -Dfile=$(PATH_DOWNLOAD)/orion.flink.connector-1.0.jar -DgroupId=org.fiware.cosmos -DartifactId=orion.flink.connector -Dversion=1.0 -Dpackaging=jar
```

Add it to your `pom.xml` file inside the dependencies section.
```xml
<dependency>
    <groupId>org.fiware.cosmos</groupId>
    <artifactId>orion.flink.connector</artifactId>
    <version>1.0</version>
</dependency>
```

## Usage
### OrionSource

* Import dependency.
```scala
import org.fiware.cosmos.orion.flink.connector.{OrionSource}
```
* Add source to Flink Environment. Indicate what port you want to listen to (e.g. 9001).
```scala
val env = StreamExecutionEnvironment.getExecutionEnvironment
val eventStream = env.addSource(new OrionSource(9001))
```
* Parse the received data.

```scala
val processedDataStream = eventStream.
    .flatMap(event => event.entities)
    // ...processing
```

The received data is a DataStream of objects of the class **`NgsiEvent`**. This class has the following attributes:

* **`creationTime`**: Timestamp of arrival.

* **`service`**: Fiware service extracted from the HTTP headers.

* **`servicePath`**: Fiware service path extracted from the HTTP headers.

* **`entities`**: Sequence of entites included in the message. Each entity has the following attributes:

  * **`id`**: Identifier of the entity.

  * **`type`**: Node type.

  * **`attrs`**: Map of attributes in which the key is the attribute name and the value is an object with the following properties:

    * **`type`**: Type of value (Float, Int,...).

    * **`value`**: Value of the attribute.

    * **`metadata`**: Additional metadata.


### OrionSink
* Import dependency.
```scala
import org.fiware.cosmos.orion.flink.connector.{OrionSink,OrionSinkObject,ContentType,HTTPMethod}
```
* Add sink to source.
```scala
val processedDataStream = eventStream.
 // ... processing
 .map(obj =>
    new OrionSinkObject(
        "{\"temperature_avg\": { \"value\":"+obj.temperature+", \"type\": \"Float\"}}", // Stringified JSON message
        "http://context-broker-url:8080/v2/entities/Room1", // URL
        ContentType.JSON, // Content type
        HTTPMethod.POST) // HTTP method
 )

OrionSink.addSink( processedDataStream )
```

The sink accepts a `DataStream` of objects of the class **`OrionSinkObject`**. This class has 4 attributes:

- **`content`**: Message content in String format. If it is a JSON, you need to make sure to stringify it before sending it.

- **`url`**: URL to which the message should be sent.

- **`contentType`**: Type of HTTP content of the message. It can be `ContentType.JSON` or `ContentType.Plain`.

- **`method`**: HTTP method of the message. It can be `HTTPMethod.POST`, `HTTPMethod.PUT` or `HTTPMethod.PATCH`.

## Production

When packaging your code in a JAR, it is common to exclude dependencies like Flink and Scala since they are typically provided by the execution environment. Nevertheless, it is necessary to include this connector in your packaged code, since it is not part of the Flink distribution.
