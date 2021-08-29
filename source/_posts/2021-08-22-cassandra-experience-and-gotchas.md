layout: post
title: 'Building Applications With Cassandra: Experience And Gotchas'
date: 2021-08-22 22:35
comments: true
categories: ComputerSystem
tags: ['Cassandra', 'Database', 'NoSQL']
---

# Experience

## Election and Paxos

When really necessary, you can leverage Cassandra's built-in Light-weight Paxos transactions for elections to determine a leader node in the cluster.

Paxos election contains three phases

See references at:

- Consensus in Cassandra: https://www.datastax.com/blog/consensus-cassandra
- Leader election with Cassandra https://www.dotconferences.com/2015/06/matthieu-nantern-leader-election-with-cassandra

<!--more-->

## Storing Time-series Data

Some optimizations you can do while saving time-series data with a deadline in mind.

https://thelastpickle.com/blog/2016/12/08/TWCS-part1.html


## Membership Change When One Node is Down

## Dynamically manipulate tables


## Deletion In Cassandra Is Hard

When not careful, Cassandra's Quorum read/write can still result in dirty data in very special cases.

Due to its design, Cassandra can have some pretty complex steps to delete data!

rule:
repair time <= gc_grace_seconds

https://thelastpickle.com/blog/2016/07/27/about-deletes-and-tombstones.html

Still, it's recommended that Cassandra cluster be constantly repaired.

https://docs.datastax.com/en/archived/cassandra/2.1/cassandra/operations/opsRepairNodesWhen.html



# References

- Cassandra Definitive Guide
- Real Time Application | Apache Cassandra | Edureka https://www.youtube.com/watch?v=CLVgy98idI0