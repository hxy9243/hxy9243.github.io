title: 'Reading Summary 2019-08'
date: 2019-08-18 22:15:57
tags: ['Reading', 'Debug', 'DistributedSystems']
categories: Reading
---

## [Cassandra Time Series Bucketing](http://msvaljek.blogspot.com/2015/11/cassandra-time-series-bucketing.html)

How to model timeseries data with Cassandra.

## [Simple GoRPC](https://github.com/ankur-anand/simple-go-rpc)

The best way to understand something, is to build one yourself. This tutorial covers basic network programming in Go, struct design and the usage of `reflect` package.

## [Optimizing M3: How Uber Halved Our Metrics Ingestion Latency by Forking the Go Compiler](https://eng.uber.com/optimizing-m3/)

A great experience sharing blog on how to debug a performance issue in their services. And with profiling and analysis tools, the Uber team was able to pinpoint this issue in worker pool and goroutine stack allocation, and then they forked the Go compiler to prove it's a regression in the Go compiler. A very nice read and analysis process.

## [Book: Programming Models for Distributed Computation](https://github.com/heathermiller/dist-prog-book)

A programming book on topics in distributed computation, from teaching experience in distributed system course, from Northeastern University.

## [Spotify Engineering Culture](https://labs.spotify.com/2014/03/27/spotify-engineering-culture-part-1/)

A very nice engineering blog from 2014. A excellent overview of Spotify culture, and an introduction on how to build the "agile" team.

## [How We Helped Our Reporters Learn to Love Spreadsheets](https://open.nytimes.com/how-we-helped-our-reporters-learn-to-love-spreadsheets-adc43a93b919?gi=26f780cc274a)

NYTimes has released its in-house course to teach journalists data science. Journalism can also benefit from a little coding/data analytics skills.
