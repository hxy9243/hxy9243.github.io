layout: post
title: 'Building Applications With Cassandra: Experience And Gotchas'
date: 2021-09-03 01:30
comments: true
categories: ComputerSystem
tags: ['Cassandra', 'Database', 'NoSQL']
---

Recently I've summarized [some experience on quickly getting started with Cassadra](/2021/08/21/17-cassandra-experience/). And for this post I'd like to introduce some of our experience and gotchas in using and operating Cassandra. Hopefully it could be useful to you, and help you avoid future unwanted surprises.

# Election and Paxos

Cassandra is always considered to be favoring the "AP" in "CAP" theorem, where it guarantees eventual consistency for availability and performance. But when really necessary, you can still leverage Cassandra's built-in Light-weight Transactions for elections to determine a leader node in the cluster.

Basically, it works by writing to a table with your own lease:

```sql
INSERT INTO leases (name, owner) VALUES ('lease_master', 'server_1') IF NOT EXISTS;
```

The `IF NOT EXISTS` triggers the Cassandra built-in ["Light-weight Transaction"](https://docs.datastax.com/en/cql-oss/3.3/cql/cql_using/useInsertLWT.html) and can be used to declare a consensus among a cluster. With a default TTL in the table, this can be used for leases control, or master election. For example:

```sql
CREATE table leases (
    name text,
    owner text,
) WITH default_time_to_live = 16; 
```

So that the lease owner needs to keep writing to the lease row for heartbeats.

I'm not sure about the performance characteristics of Cassandra's election behavior with other applications (etcd, Zookeeper, ...) and it'll be interesting to see a study. But since those are already more full-featured and well-understood in keeping consensus, I'd recommend delegating this behavior to them unless you're stuck with one database.

<!--more-->
# Optimizing Time-series Data Retention

One great use case of Cassandra is logs and timeseries data saving. But what if you'd want to automatically drop stale data and don't want to populate the tombstones in Cassandra? Removing and updating data frequently [may actually cause problems](https://www.instaclustr.com/support/documentation/cassandra/using-cassandra/managing-tombstones-in-cassandra/#section-when-do-tombstones-cause-problems) in Cassandra.

Cassandra team developed a very useful strategy to just handle this situation. It's called TWCS (Time Window Compaction Strategy). And it works by grouping your timeseries data into chunks (in the same SSTable) and directly dropping them when their TTL is reached, instead of generating new tombstones. Check out [this blog](https://thelastpickle.com/blog/2016/12/08/TWCS-part1.html) for use cases and details.

So that you can create a table with these flags enabled:

```sql
-- creating table compacting data every day, with 7 days TTL and TWCS
CREATE TABLE timeseries (
    ...
) WITH CLUSTERING ORDER BY (value ASC)
    AND gc_grace_seconds = 60
    AND default_time_to_live = 604800  
    AND compaction = {
        'compaction_window_size': '24', 
        'compaction_window_unit': 'HOURS', 
        'class': 'org.apache.cassandra.db.compaction.TimeWindowCompactionStrategy'}
```

It's some neat optimizations you can do while saving time-series data with a deadline in mind.

# Membership Change When One Node Is Down

Interestingly enough, Cassandra can get grumpy when you try to man-handle its membership. For example, during our development and testing, we encountered this issue where the cassandra cluster is just reluctant to accept a new node when there's already a node down. The logs from the node shows:

```
CassandraDaemon.java:465 - Exception encountered during startup
java.lang.RuntimeException: A node required to move the data consistently is down (/x.x.x.x).  
If you wish to move the data from a potentially inconsistent replica, restart the node with -Dcassandra.consistent.rangemovement=false
```

It turns out that Cassandra needs to move the data consistently to the new node. And when one node is down and Cassandra cannot form a quorum for the data with one node missing, it'll be reluctant to hand the potentially broken data to the newcomer.

Here's also an interesting [blog](https://medium.com/analytics-vidhya/replacing-a-dead-node-in-cassandra-and-surprises-4681287eeddf) about replacing Cassandra dead node and all the surprises along the way. The lesson is: managing Cassandra membership could be harder than you actually thought. So it might be a good idea to read the manual.

# Dynamically Manipulating Tables Is Bad

When we started building our application, we used a way to automatically create new tables. It worked well for a while, and then we kept hitting this weird error:

```
Caused by: org.apache.cassandra.exceptions.ConfigurationException: 
  Column family ID mismatch (found <SOME UUID-1>; expected <SOME UUID-2>)
```

It turns out Cassandra long had this problem with running into race conditions with creating Column Families (a.k.a Cassandra's tables).

- https://stackoverflow.com/questions/64410561/how-to-resolve-column-family-id-mismatch-error
- https://stackoverflow.com/questions/68979637/getting-around-column-family-id-mismatch-exception-when-i-retry-the-query-while

After searching through the Internet, our conclusion is simply: do not attempt to dynamically create tables in a distributed system in the first place. We redesigned our application and schema and this problem went away since.

# Deletion In Cassandra Is Hard

It's not from our own experience, but I still feel like it's worth sharing. When not careful, Cassandra's Quorum read/write can still result in dirty data in very special cases. Due to its design, Cassandra can have some pretty complex steps to delete data!

https://thelastpickle.com/blog/2016/07/27/about-deletes-and-tombstones.html

Rule of thumb from this experience: repair time <= gc_grace_seconds. So that repair would propagate tombstones before GC cleans it up. Still, it's recommended that Cassandra cluster be constantly repaired.

https://docs.datastax.com/en/archived/cassandra/2.1/cassandra/operations/opsRepairNodesWhen.html

Here's another interesting case for deletion in Cassandra causing headaches and surprises, due to tombstone hurting performance. It's from [Discord's Experience](https://blog.discord.com/how-discord-stores-billions-of-messages-7fa6ec7ee4c7):

> We noticed Cassandra was running 10 second “stop-the-world” GC constantly but we had no idea why. We started digging and found a Discord channel that was taking 20 seconds to load.
> ...
> To our surprise, the channel had only 1 message in it. It was at that moment that it became obvious they deleted millions of messages using our API, leaving only 1 message in the channel.
> If you have been paying attention you might remember how Cassandra handles deletes using tombstones (mentioned in Eventual Consistency). When a user loaded this channel, even though > there was only 1 message, Cassandra had to effectively scan millions of message tombstones (generating garbage faster than the JVM could collect it).

Basically if you are not careful, deletion in Cassandra could actually be a part of the query burden due to the tombstones. Understanding Cassandra's behavior is essential to operation at its best performance.

# References

- Consensus in Cassandra: https://www.datastax.com/blog/consensus-cassandra
- Lightweight Transactions in Cassandra: https://docs.datastax.com/en/cassandra-oss/3.0/cassandra/dml/dmlLtwtTransactions.html
- Leader election with Cassandra https://www.dotconferences.com/2015/06/matthieu-nantern-leader-election-with-cassandra
- Time Window CompactionStrategy https://cassandra.apache.org/doc/latest/cassandra/operating/compaction/twcs.html
-  TWCS part 1 - how does it work and when should you use it ?  https://thelastpickle.com/blog/2016/12/08/TWCS-part1.html
- Replacing a dead node in Cassandra and surprises https://medium.com/analytics-vidhya/replacing-a-dead-node-in-cassandra-and-surprises-4681287eeddf
- StackOverflow https://stackoverflow.com/questions/28376437/how-to-recover-cassandra-node-from-failed-bootstrap/28379751
- Manual For Adding/Removing Node: https://docs.datastax.com/en/cassandra-oss/2.1/cassandra/operations/opsAddingRemovingNodeTOC.html

