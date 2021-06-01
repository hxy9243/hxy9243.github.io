layout: post
title: 'Golang Channel Idioms'
date: 2021-05-31 18:51
comments: true
categories: Golang
tags: ['Learning', 'Golang', 'Channel']
---

While learning Golang, I was fascinated with the power Golang's goroutines and channels.
Channel is a powerful tool to tackle synchronization problems
in asynchronous programs. It acts as a bridge between async goroutines and can describe 
some complicated logic expressively.
Together they can be powerful weapons in building async applications. On the other hand, when misused, they can be a nightmare to debug.

Here I've summarized a few of the valuable idioms of using Golang routines from multiple references as well as my own experience. They can serve as a helpful toolbox that comes in handy for similar problems. 
So that you don't have to design them from scratch, which might help you avoid synchronization errors.

<!--more-->

For more information on Golang channels, see links in the [references](#references). 
They should give you a good introduction to channels and their
basic behaviors.

If you found any problems or some idioms that you think this experience summary can provide. 
Please feel free to message me and let me know. Many thanks in advance!

I suggest you read the description and implement a toy version of each of these idioms first. 
Building your own implementation serves as a good exercise.

# 1. Use Channels To Get Results

## 1.1. Get Results From Async Functions

This is a very common usage of goroutines: to get results back from multiple async goroutines. Child goroutine can return a channel that finally sends back the result, which is akin to the Future/Promise idiom in some other languages.

Exercise: How to create an example, where you spawn multiple goroutines with work asynchronously (emulated with `time.Sleep()`), and then collect and print the results in the main thread?

See my implementation [here](https://github.com/hxy9243/go-examples/blob/main/src/channels/get_result.go).

## 1.2. Future-like implementation

You can also emulate Future/Promise. Functions can return channels wrapped in Futures, so as to process and return results asynchronously.

See my implementation [here](https://github.com/hxy9243/go-examples/blob/main/src/channels/future.go).

## 1.3. Iterate through all results from channel

A processor can be put in a goroutine and keep getting results until it's closed.

```golang
for r := range mychannl {
  // process
}
```

The idiom is for the sender to close the channel. to avoid race conditions where sender sends on a closed channel. Sending on a closed channel will cause panic!

Exercise: how to create an example where you use the "for-range" idiom to get all results from the worker threads?

Hint: you'll need proper way to make sure all worker threads are done with sending to the channel and close it properly. Consider `sync.WorkGroup`.

See my implementation [here](https://github.com/hxy9243/go-examples/blob/main/src/channels/iterate_channel.go)

# 2. Use Channels For Timeouts, Signals, and Cancellations

## 2.1. Handling Context

For network requests, exec commands, Golang actually has a great mechanism on setting timeouts and cancelling: `context`.

See more at: https://golang.org/pkg/context/

It is actually my recommended way of handling cancelling and timeouts in requests. It has a neat and clean interface, easier to use, and less error-prone to home-brew solutions with channels.

For example, golang's `cmd/exec.Command` can be cancelled either by user or timeouts. 
The underlying implementation listens to `context.Done()` 

See: https://golang.org/src/os/exec/exec.go?s=6768:6841#L394

What if you'll need to implement your own requests that takes in a context and respect the cancelling signal? You can learn from the example above in Golang library.

My example of [handling context](https://github.com/hxy9243/go-examples/blob/main/src/channels/context.go).

## 2.2. Timeouts and Tickers.

You can use `time` package to handle timeouts, periodic events. 
Golang's `time` package uses channels for callbacks, which makes logic easier to read and understand.

Exercise: how to create an example that uses timer/ticker to trigger/cancel an action?

My example [here](https://github.com/hxy9243/go-examples/blob/main/src/channels/timer.go).

## 2.3. Processing OS Signals

You can use channels for notifications. One example is OS signals.
Instead of installing signal handlers like some other languages, Golang's signal package actually returns a channel. So users can handle channels like any other channels.

Notice:

From Go documentation (https://golang.org/pkg/os/signal/#Notify)

Package signal will not block sending to c: the caller must ensure that c has sufficient buffer space to keep up with the expected signal rate. For a channel used for notification of just one signal value, a buffer of size 1 is sufficient. 

```
// Set up channel on which to send signal notifications.
// We must use a buffered channel or risk missing the signal
// if we're not ready to receive when the signal is sent.
```

Exercise: how to create an example of using OS signals to cancel current work?

My example [here](https://github.com/hxy9243/go-examples/blob/main/src/channels/signal.go).

## 2.4. Use Channels As Message Broker

Channels can be used as a message broker when you need to send messages to multiple child goroutines.

It's a kind of like the reverse of handling multiple message sources. The parent channel can send signal to each one of the channels that's accepted by each child goroutine.

Exercise: how to create an example where child workers start after parent broadcasts a ready signal?

My implementation [here](https://github.com/hxy9243/go-examples/blob/main/src/channels/message_broker.go).

# 3. Use Channels For Rate Limiting

## 3.1. Acts Like Semaphore For Limiting Worker Rate

```golang
  // init semaphore
  sem := make(chan struct{}, 4)
    for i := 0; i < 4; i++ {
    sem <- struct{}{}
  }

  // run the actual work, taking one from semaphore before
  // starting, and give back once work is done
  go func() {
    <- sem
    defer() {
      sem <- struct{}{}
    }()
	  // some work here
  }()
```

Exercise: how to use channel as a semaphore that limits parallel processing to a given rate? i.e. Limit to at most N workers processing at the same time? 

My example [here](https://github.com/hxy9243/go-examples/blob/main/src/channels/semaphore.go).

(How is this implementation better than evenly dividing the work to multiple goroutines before work starts? e.g. dividing 16 works to 4 goroutines, each processes 4 works?)

## 3.2. Competing consumers

Similarly, you can solve the above examples with competing consumers. Start N consumers from the start, and all of them take from the same channel.

My example [here](https://github.com/hxy9243/go-examples/blob/main/src/channels/consumers.go).

# 4. Other Interesting Use Cases:

- Rate Limiting (Leaky bucket like implementation) with Go channels: https://github.com/golang/go/wiki/RateLimiting
- Pubsub with Go channels: https://eli.thegreenplace.net/2020/pubsub-using-channels-in-go/

# References

- Golang Tour: https://tour.golang.org/concurrency/2
- Golang context: https://golang.org/pkg/context/
- Go101 on Channels: https://go101.org/article/channel.html
- Go101 on Channel Use Cases: https://go101.org/article/channel-use-cases.html
- Go Pubsub implementation in Channels: https://eli.thegreenplace.net/2020/pubsub-using-channels-in-go/
- Leaky Bucket implementation with Go channels: https://github.com/golang/go/wiki/RateLimiting
- The Behavior Of Channels: https://www.ardanlabs.com/blog/2017/10/the-behavior-of-channels.html


