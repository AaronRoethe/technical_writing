# Event-Driven Design

## Summary

Event-Driven Design (EDD) is a software architecture pattern that emphasizes the use of events as the primary means of communication and coordination between different parts of a system. In EDD, events are used to signal changes or important occurrences within a system, and are passed from one component to another as needed.

EDD can be useful in a variety of contexts, including distributed systems, microservices architectures, and real-time data processing systems. By using events as the main communication mechanism, EDD can help decouple different components of a system, making it easier to scale, maintain, and modify over time.

One of the key benefits of EDD is its ability to support asynchronous processing. Because events can be produced and consumed independently of each other, EDD systems can be designed to handle large volumes of events with low latency and high throughput.

To implement EDD, systems typically use an event bus or message broker to manage the flow of events between different components. Components can then subscribe to or publish events as needed, based on their specific roles and responsibilities within the system.

Overall, EDD is a powerful architecture pattern that can help simplify and scale complex systems by emphasizing the use of events as the primary means of communication and coordination.

# Design Details

## Debezium For Event Propagation
Debezium is an open-source distributed platform that provides change data capture (CDC) capabilities for streaming data from database systems. With Debezium, developers can monitor changes to data in a variety of databases and send those changes to other systems in real-time, allowing for powerful data integration and analysis.

* Connectors for many source databases (e.g. MySQL, Cassandra, SQL Server, Vitess)
* Supports “many identical databases” use case:
    * [Topic Routing](https://debezium.io/documentation/reference/1.4/configuration/topic-routing.html) - logical tables
        * [Kafka Connect SMT](https://kafka.apache.org/documentation/#connect_transforms)
        * [Content-based routing](https://debezium.io/documentation/reference/1.4/configuration/content-based-routing.html)
        * [Decrease memory consumption: database-per-tenant pattern](https://debezium.io/documentation/faq/#how_to_decrease_memory_consumption_in_a_database_per_tenant_pattern)

* OOB integration with OpenTracing/Jaeger and Prometheus/Grafana.
* Testcontainers support
* [Example](https://github.com/debezium/debezium-examples/tree/main/topic-auto-create)

## Kafka 
Apache Kafka is an open-source distributed streaming platform that is used for building real-time data pipelines and streaming applications. Kafka provides a distributed, fault-tolerant, and scalable platform for handling large volumes of data streams in real-time.

* Design Pattern [cloudevents](https://github.com/cloudevents/spec)
    * supported by [Debezium](https://debezium.io/documentation/reference/1.4/integrations/cloudevents.html)
* Protocol buffers message types
    * A language-agnostic binary serialization format used to efficiently serialize and deserialize structured data.
* Consumers should be able to read any event
    * Promoting: flexibility, future-proofing, reusability, scalability

Stream Types
* Close Source Data: `Raw`
* Public Message Structure: `consistency contract`

Topics
* priv_service
* service_events

Partitioning
In Kafka is the process of splitting a Kafka topic into multiple smaller "partitions." Each partition is an ordered and immutable sequence of records, which can be thought of as a "log" of messages. When a producer publishes a message to a Kafka topic, the message is appended to the end of the log in the appropriate partition.

Consumers
The framework should provide:

* Easy deployment & configuration in Kubernetes.
* Easy access to known event message types.
* Configurable retry logic.
* Easy configuration to skip uninteresting messages.


## Metrics
* [Debezium integration](https://debezium.io/documentation/reference/1.4/integrations/tracing.html)

The end state is to see both latency and volume-based metrics for:

1. Raw event rows being pushed into Kafka.
2. Raw events being consumed and transformed into public messages.
3. Public messages being pushed into Kafka.
4. Consumer retrieval and processing of public messages.
5. Errors, retries at every level.

General Kafka metrics
* request and error rates
* under-replicated partitions and offline partition count
* total broker partitions
* log flush latency
* consumer message rate
* consumer max lag
* free memory and swap space usage

## Bottlenecks

Consumers/Partitioning Scheme
Consumer group rebalancing: When a consumer joins or leaves a consumer group, Kafka needs to rebalance the partitions assigned to each consumer to ensure that the load is evenly distributed. This process can take some time, especially if the number of partitions is large or if there are many consumers in the group.

Consumer lag: Consumer lag occurs when a consumer is unable to keep up with the rate of messages being produced by a producer, resulting in a backlog of messages in the partition. This can be caused by slow processing or network latency, and can lead to delayed or inconsistent processing of messages.

Data skew: Data skew occurs when some partitions in a topic have significantly more messages than others, leading to imbalanced processing and potential bottlenecks. This can be caused by uneven distribution of messages from producers or uneven processing by consumers.

Overloaded partitions: Overloaded partitions can occur when too many messages are produced to a single partition, causing processing delays and potentially impacting the performance of other partitions. This can be caused by high message throughput or imbalanced partitioning strategies.

Disk and network I/O: Kafka relies heavily on disk and network I/O to process messages, and slow disk or network performance can lead to bottlenecks in both producer and consumer performance.