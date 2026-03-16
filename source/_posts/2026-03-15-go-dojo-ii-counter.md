title: Golang Dojo Project II - Distributed Counter
date: 2026-03-15 17:45:08
categories: [Golang, ComputerSystem]
tags: ["opensource", "programming", "computer system"]
---

# Introduction

I recently took a stab at a scalable distributed counter implementation. The inspiration comes from an common System Design interview question: "Design a distributed counter that can handle millions of events per second and provide near real-time read access to the current count."

I read some about it in theory and design interview questions, but I never actually see it in action. Thanks to the advance of AI, a lot of the implementation burden can be handed off to an AI agent, while I focus more on the overall design.

So now I added it to the Golang Dojo project as I thought it could be a fun and education project to deepen my understanding of distributed systems, and also to explore how AI can assist in such process.

Source code is at: [https://github.com/hxy9243/go-dojo/tree/main/src/counter](https://github.com/hxy9243/go-dojo/tree/main/src/counter)

<!--more-->

# Problem Break-Down

The problem can be broken down into several key requirements:

- **High Write Throughput**: The system must be able to handle a very high volume of increment events (millions per second), which means it needs to be horizontally scalable and efficient at handling concurrent writes.
- **Fault Tolerance**: The system should be resilient to failures, ensuring that increments are not lost and that the count remains accurate even in the face of node failures or network issues.
- **Scalability**: The system should be able to scale horizontally to accommodate increasing load, both in terms of write throughput and read access.
- **Latency**: The system should provide near real-time read access, with a delay of a few seconds. This means that the read path should be optimized for low latency, even under high load.

An interesting part of the design is the data consistency model.

For a counter, we can often tolerate eventual consistency for higher availability and lower latency. This allows us to use distributed systems that prioritize availability and partition tolerance while still providing a reasonably accurate count.

There's still a question of accuracy vs availability.

To achieve the requirements of the system, we need to consider the following areas of design.

## Write Path Design

The write path of the system can be designed with the following requirements in mind:

1. To decouple the write API from the database, allowing for asynchronous processing and better handling of spikes in traffic, providing better throughput under high loads.
2. To enable horizontal scalability of the write path.
3. To provide durability and fault tolerance for incoming events.

Kafka is a natural choice for the streaming component of the system. It provides a distributed, partitioned streaming service the decouples the write API from writing to the database.

The topic design is: we use a single topic called `counter` with multiple partitions (e.g., 3 in this toy example). Each increment event is published to Kafka with the `eventkey` as the message key. In real-world production systems, these would likely be to be tuned.

## Aggregation and Batching

After the events are published to Kafka, we need a component that consumes these events, aggregates them in memory.
They'll need to periodically flush the aggregated counts to the database. This allows us to reduce the number of write operations to the database, which can be a bottleneck.

They'll also need to be independently scalable and fault tolerant. They could run multiple replicas, each reading from different partitions of the Kafka topic.

## Backend Database

The backend database needs to be able to handle high write throughput, fault tolerant, and scalable, to store the aggregated counts.

A distributed NoSQL database like Cassandra comes to mind.

I've used Cassandra's `counter` data type to efficiently store and update the counts, built specifically for this use case.

**Notice:** The trade-off of using this data type is that it can lead to over-counting in edge cases (e.g., if the aggregator crashes after updating the counter but before updating the offset). However, for many use cases (like counting views or likes), this level of precision could be acceptable in exchange for high availability and performance.

The alternative would be to use a locking mechanism to ensure strict consistency, but this will lead to more compute and latency overhead, especially under high write loads. It really depends on the specific use case and requirements.

## Read API Design

The read API needs to provide a low-latency, highly available interface for retrieving the current count for a given key. To achieve this, we can use a caching layer (like Redis) in front of the database to serve read requests quickly.

The read API will first check the cache for the count value. We'll also need to tune the cache TTL to balance between read performance and data freshness.

# Architecture Design Overview

With these considerations in mind, the overall architecture of the system can be visualized as follows:

```text
    [Write API Worker] --> [Kafka] --> [Aggregator Worker] --+
                                                              |
                                                              v
                                                      [Backend Database]
                                                              ^
                                                              |
    [Read API Worker] --> [Cache] ----------------------------+
```

The counter application is a distributed, event-driven system designed to handle high-throughput counting operations. It is split into three main components:

1. **Write API Worker (`write-api`)**:
    An HTTP server that accepts count increments via a `POST /event` endpoint. It acts as a producer, taking incoming requests and publishing them as messages to a Kafka topic.

    The write API can instantly acknowledge write requests without waiting for a slow database transaction. This acts as a shock absorber during traffic spikes.
2. **Kafka Streamer**:
    A Kafka topic (`counter`) that serves as a durable, partitioned log of increment events. It decouples the write API from the database, allowing for asynchronous processing and better handling of traffic spikes.
3. **Aggregator Worker (`consumer`)**:
    A background worker that consumes messages from the Kafka topic. It aggregates the increments in memory and periodically flushes the aggregated counts to a persistent database (Cassandra).
4. **Backend Database (Cassandra)**:
    A distributed NoSQL database that stores the aggregated counts. It uses Cassandra's `counter` data type for efficient counting operations.

    **Tables**:
    - `partitions (partition int PRIMARY KEY, offset bigint)`: Stores the latest processed Kafka offset for each partition.
    - `counters (key text PRIMARY KEY, counter counter)`: Stores the actual aggregated counter values using Cassandra's native `counter` data type.

    This way the aggregator can track which messages have been processed and provide some level of deduplication in case of consumer restarts or rebalances.
5. **Read API Worker (`read-api`)**:
    An HTTP server that serves the current count for a given key via a `GET /counter` endpoint. It uses Redis as a caching layer to serve high-volume read requests quickly, falling back to Cassandra when the cache is empty.

    Redis stores the counter values as basic key-value pairs with a TTL (Time-To-Live). Also uses a `.counter-lock` key for distributed locking.
    Since exact up-to-the-millisecond precision is often not critical for counters, caching the value for a short TTL (e.g., 500ms) saves Cassandra from heavy read loads.

# Implementation Details

- **Cache Stampede Prevention**: In the Read API, if a cache miss occurs, the worker attempts to acquire a distributed lock in Redis (`SETNX` on `key.counter-lock`). Only the worker that acquires this lock will query Cassandra to fetch the latest value and populate the cache. Other concurrent requests will wait and retry reading from the cache. This prevents the "thundering herd" problem where a cache miss on a hot key causes a massive spike in database queries.
- **Exactly-Once Semantics (Deduplication)**: The Aggregator manually tracks Kafka partition offsets in the `partitions` Cassandra table. Before processing a message, it checks if the message's offset is less than or equal to the stored offset. This mechanism helps skip redelivered messages, providing exactly-once (or deduplicated) processing guarantees under normal conditions.
- **Graceful Shutdown and Rebalancing**: The Aggregator listens for Kafka partition rebalance events (`kafka.RevokedPartitions`) and system signals (SIGINT/SIGTERM). When a partition leaves the consumer or the program stops, it flushes any pending in-memory counts to Cassandra to ensure no data is lost.

# Deployment

I've created a Helm chart to deploy the application on Kubernetes. The chart includes deployments for the Write API, Aggregator, and Read API workers, as well as services to expose the APIs.

Each component can be scaled independently based on the load. For example, during high write traffic, we can scale up the Write API workers, while during high read traffic, we can scale up the Read API workers.

See more at: https://github.com/hxy9243/go-dojo/tree/main/src/counter/helm/counter-app

# Future Work

Alas, this project is meant for my own exploration and education. And it isn't a production-ready system. There are many areas for improvement and further development

- **In-Memory Map Bounding for Aggregator**: The Aggregator's in-memory `Counters` map could grow unboundedly if there's a huge surge of unique keys within the 500ms flush window. Adding a maximum capacity and an LRU eviction policy to force early flushes would protect the worker from out-of-memory errors.
- **Idempotency Edge Cases**: Updating the Cassandra `counter` and `partitions` offset happens in the same execution flow but not in an isolated, fully ACID transaction. If the program crashes exactly between updating the `counters` table and the `partitions` table, a message could be re-processed upon restart, slightly over-counting.
- **Saving historical time-series data**: The current design only keeps the latest count for each key. If we wanted to keep a history of counts over time (e.g., for analytics or trend analysis), we would need to modify the database schema and aggregation logic to store time-series data.
- **Adjust partitions**: The number of Kafka partitions can be tuned based on the expected load and the number of consumer instances. More partitions can allow for higher parallelism but also increase complexity in managing offsets and ensuring order.
- **Testing and Validation**: Implementing comprehensive testing, including unit tests, integration tests, and load testing, would be crucial to ensure the reliability and performance of the system under various conditions.
- **Auto-Scaling**: Implementing auto-scaling for the different components based on metrics like CPU usage, memory usage, or request latency would help maintain performance during traffic spikes. This could be achieved using Kubernetes Horizontal Pod Autoscaler (HPA).
- **Monitoring and Alerting**: Implementing monitoring for Kafka consumer lag, database performance, and API response times would be crucial for maintaining the health of the system. Tools like Prometheus and Grafana could be used for this purpose.

This is an educational project so there might be some design and implementation choices that are not optimal for a production system.

If you've read this far, please let me know if you have any suggestions or feedback on the overall design and implementation!

# Summary

Over the course of this small project, I've started with an idea in mind, and had a long conversation with ChatGPT over the design, implementation of this system. I've learned much about from AI by simply providing a direction. Also AI has freed me from the burdens of creating boilerplate (like using the library and creating Helm for k8s deployment). In the process I've also learned about many implementation details on installing and using the libraries for Kafka. This turned out to be valuable learning experience developing with AI assistance.

With the advancement of far more autonomous AI coding agents, software development is going to be more focused on higher-level system architecture understanding, trade-offs discussions.
So I'd explored more with this Dojo way of learning systems with Golang. As I believe a thorough understanding of the underlying system design and architecture is still crucial in the age of AI. The future is about learning with AI, instead of replacing all understanding.

Source code is at: [https://github.com/hxy9243/go-dojo/tree/main/src/counter](https://github.com/hxy9243/go-dojo/tree/main/src/counter)

# References

- https://netflixtechblog.com/netflixs-distributed-counter-abstraction-8d0c45eb66b2
- https://systemdesign.one/distributed-counter-system-design/
- https://docs.confluent.io/kafka/introduction.html
