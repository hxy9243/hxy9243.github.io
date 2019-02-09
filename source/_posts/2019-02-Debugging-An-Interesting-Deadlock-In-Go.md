---
title: Debugging An Interesting Deadlock in Golang
date: 2019-02-09 14:05:32
tags: ['Golang', 'Mutex', 'Channel']
---

This week I've been chasing a deadlock issue in a Golang server application, which will essentially render the server unresponsive to client requests indefinitely and cannot recover in anyway without restarting. I've trying all ways days and nights, even ended up re-writing a small portion of the application to clean up all the locks - no luck.

<!-- more -->

## Root Cause

The root cause of this vexing issue is the combination use of mutex locks and blocking channels. In Golang, channels are also used often as a powerful way for sychronization. They're often used to protect inner states of a structure, or to distribute workloads, to make sure different actions are not taken at the same time.

See here: https://medium.com/stupid-gopher-tricks/more-powerful-synchronization-in-go-using-channels-f4a1c3242ed0

```go
go func() {
  for {
    select {
    case value = <-h.setValCh: // set the current value.
    case h.getValCh <- value: // send the current value.
  }
}()
```

By using a big select statement as a mux for all coming read and write requests, channels protect shared states, just like mutexes, and sometimes with more flexibility (e.g. when you include timer or ticker in the code). However it could be dangerous when people don't realize, as a way of synchronization, channels are as well as prone to misuse, especially when mixed with mutexes.

Here's an example of misusing channels to cause an deadlock. See if you can spot it:

```go
func foo() {
	a := make(chan bool)
	b := make(chan bool)
	done := make(chan bool)
	go func() {
		for {
			select {
			case <-a:
				fmt.Println("case A")
				<-b
			case <-b:
				fmt.Println("case B")
			case <-done:
				fmt.Println("case done")
				break
			}
		}
	}()
}
```

It could be easy to reason about deadlocks when you're using mutexes only, or when you're using channels only, but perhaps not so easy when you're mixing both.

Below is the simplified version of the deadlock bug, demonstrating how mutexes and channels used together can cause interesting issues. Without reading further can you spot the issue?

```go
type A struct {
    mtx *sync.Mutex
    // other data structures
}

type B struct {
    action chan bool
    clear  chan bool
    // other channels and data structures
}

a := NewA()
b := NewB()

func NewB() *B {
    go func() {
        for {
            select {
            case <- clear:
                // clear records
            case <- action:
                a.Action()
                // ... other cases
            }
        }
    }()
    // other initializations
}

func (a *A) Action() {
    a.Mtx.Lock()
    defer a.Mtx.Unlock()

    // do action
}

func (a *A) Foo() {
    a.Mtx.Lock()
    defer a.Mtx.Unlock()

    // do some other actions
    b.clear <- true
}
```

The issue lies where `Action()`, and `Foo()` can be called simultaenously or in very close time, and they both enter the critical section of `A`'s mutex locks. And `B`'s mux uses blocking channels to coordinate different actions, the `b.clear <- true` statement will block if code in previous case has not been completed.

Therefore, `a.Action()` and `a.Foo()` can both be locked, and `b.clear` is blocked as it's waiting for `a.Action()` to finish, which is not going to happen when `a.Action()` is waiting for `a.Foo()` to unlock!

## Useful Debugging Tools

In debugging experience I haven't run into a very good tool that'll analyze this type of deadlock. There are several tools that deals with mutex locks only. There's one even built inside Golang's runtime, but that's not enough, as it only detects if all the goroutine are locked.

I've used `gdb` and Golang's `pporf` library. The convenience of `pprof` library is that, if you're writing a server application, you can directly register an HTTP endpoint with all useful debug output on `/debug/pprof`. The one I used dumped all the running goroutines in the application:

```bash
curl localhost:10000/debug/pprof/goroutines?debug=1
```

And when examining all the outputs when a deadlock happens, you need to pay attention to the following details:

- What mutex lock are still pending. As they're competing with the locks, and can potentially be the culprit that contributed to the deadlock.
- What channels are pending. This could be hard and easy to omit, as there can be a lot of channels that are pending by design: they are waiting for signals for certain actions, not necessarily out of a deadlock. So, it might be faster to start examining channels used in the sychronizing channels pattern mentioned above.

On a side note, the `pprof` can be really useful if you're trying to understand how the program is behaving. I even identified a resource leak in the code using `pprof` (maybe I'll write another blog to discuss it). See more at:

- https://golang.org/pkg/net/http/pprof/
- https://jvns.ca/blog/2017/09/24/profiling-go-with-pprof/
- https://blog.minio.io/debugging-go-routine-leaks-a1220142d32c

Example goroutine output from `pprof`, from [blog](https://blog.minio.io/debugging-go-routine-leaks-a1220142d32c) mentioned above:

```
goroutine 149 [chan send]:
main.sum(0xc420122e58, 0x3, 0x3, 0xc420112240)
        /home/karthic/gophercon/count-instrument.go:39 +0x6c
created by main.sumConcurrent
        /home/karthic/gophercon/count-instrument.go:51 +0x12b

goroutine 243 [chan send]:
main.sum(0xc42021a0d8, 0x3, 0x3, 0xc4202760c0)
        /home/karthic/gophercon/count-instrument.go:39 +0x6c
created by main.sumConcurrent
        /home/karthic/gophercon/count-instrument.go:51 +0x12b

goroutine 259 [chan send]:
main.sum(0xc4202700d8, 0x3, 0x3, 0xc42029c0c0)
        /home/karthic/gophercon/count-instrument.go:39 +0x6c
created by main.sumConcurrent
        /home/karthic/gophercon/count-instrument.go:51 +0x12b
```

## Lesson Learned

It's easy to overlook channels as a powerful synchronization tool in Golang, and bad consequences may happen. Instead of expecting deadlock tools to come and save the day, it might be more efficient to reason about the code more prudently, with the following lessons in mind:

- Channels can be used for synchronizations as well.
- Beware when you're using channels and mutexes at the same time. Reason it well! The key is not to put the blocking channel send/receive inside a critical section.
- Keep mutex protected sections as small as possible, right around the values you're trying to protect if possible. You can even consider using getter/setter for structs with protected fields, and not expose mutexes as public. This will give you much better time when you're reasoning with the code.
