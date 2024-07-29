title: |
    Paper Reading: Understanding Real-World Concurrency Bugs in Go
date: 2019-03-04 00:46:03
comments: true
tags: ['PaperReading', 'Golang', 'Concurrency']
categories: Paper
---

Link: https://golangweekly.com/link/59972/b208593eda

A team from Penn State University and Purdue published their latest study on concurrency bugs found in Golang projects, namely large projects from Github: Docker and Kubernetes, two datacenter container systems, etcd, a distributedkey-value store system, gRPC, an RPC library, and CockroachDB and BoltDB. The authors searched commit histories of each repository to understand concurrency bug fixes for categorization and study.

__TL;DR__:

- Go's message-passing concurrency mechanism, something Go is proud of, isn't as easy to use as it's generally perceived. It creates just as many bugs, if not more, than shared-memory concurrency model.
- Shared memory synchronization is still used more in Go projects.
- Go's built-in race and deadlock bug detection library still cannot catch all the bugs. There's room for more improvements.

<!-- more -->

Abstract: The author of this paper analyzed 171 bugs in 6 aforementioned open-source Go projects for a systematic study of Go concurrency bugs, providing better understanding for go bugs and concurrency bug detection tools.

## Type of Bugs

The author categorized the bugs into __blocking__ and __non-blocking__ bugs. __Blocking__ bugs are misuse of synchronization primitives that causes the program, or a subset of goroutines to hang. __Non-Blocking__ bugs happen when shared memory is unprotected, causing data races, or errorneous message passing, e.g.: when goroutines don't quit properly, causing resource leaks.

### Blocking bugs

The paper further divided blocking bugs into traditional shared memory bugs, and bugs caused by misuse of message passing, or libraries related to messaging.

This led to an interesting observation from this paper: contrary to common belief, message passing are potentially more likely to cause blocking bugs than shared memory.

An example of blocking bugs related to message passing, with its fix. similar to the one I had before:

```go
  // goroutine 1

  func goroutine1() {
      m.Lock()
-     ch <- request // blocks
+     select {
+         case ch <- request
+         default:
+     }
      m.Unlock()
  }
```

```go
  // goroutine 2
  func goroutine2() {
      for {
          m.Lock()   // blocks
          m.Unlock()
          request <- ch
      }
  }
```

An example for blocking bug related to messaging library from the paper is `Pipe` library.

The paper also noticed that for blocking bugs, there's a high correlation between blocking bugs (shared memory as well as message passing) to their fixes, indicating there's high potential in developing automated tools to help fix such bugs.

### Non-Blocking bugs

For non-blocking bugs, the paper also divided them into traditional bugs, (e.g. unprotected shared memory causing data races), misuse of channels, or shared data in special libraries.

An interesting example related to non-blocking bug caused by message passing, mentioned in the paper:

```go
// when multiple goroutines execute the following code, default
// can execute multiple times, closing the channel more than once,
// which leads to panic in Go runtime

- select {
-     case <- c.closed:
          // do something
-     default:
+         Once.Do(func() {
              close(c.closed)
+         })
- }
```

Example regarding non-blocking bug related to library, the paper mentioned the `context` library, where `context` object type is designed to be accessed by mulitple goroutines. And accessing string type in the `context` library could potentially lead to data races.

The paper observes some traditional data race detector cannot detect all types, calling for future researches on this topic.

## More Discussions

More discussions from HackerNews: https://news.ycombinator.com/item?id=19280927
