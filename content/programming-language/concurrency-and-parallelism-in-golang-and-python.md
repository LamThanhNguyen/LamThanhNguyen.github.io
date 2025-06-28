---
description: "Deep Dive Into Concurrency and Parallelism in Golang and Python"
featured_image: "/images/concurrency-parallelism-golang-python.png"
tags: ["python", "golang", "concurrency", "parallelism"]
title: "I. Concurrency and Parallelism In Golang And Python"
---

# 1. Introduction
## - Why concurrency and parallelism matter:
Modern software isn’t just about doing one thing at a time. From high-traffic web servers and real-time analytics, to background processing and automation, today’s applications often need to handle multiple tasks “at once.” Whether it’s serving thousands of user requests, processing large datasets, or running background jobs, your program’s ability to efficiently juggle several operations can be the difference between fast, scalable software and a sluggish, unresponsive system.

That’s where **concurrency** and **parallelism** come in. Mastering these concepts can lead to:
- **Faster applications** that take full advantage of modern multi-core CPUs.
- **Scalability**, allowing your services to handle more work without major rewrites.
- **Cleaner codebases**, with logic separated into manageable, independent pieces.

But while they’re often mentioned together, concurrency and parallelism are **not the same thing**.

## - Key differences between concurrency and parallelism:
**Concurrency** is about dealing with lots of things at once—structuring your program so it can manage many tasks that may need to happen together.

**Parallelism** is about doing lots of things at exactly the same time.

**The Chef and the Kitchen**

Imagine a single chef in a kitchen:
- The chef has several dishes to prepare.
- They might chop vegetables for one dish, then switch to boiling pasta for another, then check the oven for a third dish, switching back and forth as needed.

This chef is working **concurrently**: handling multiple tasks at once by rapidly switching between them, but only ever doing one thing at any given moment.

Now imagine a *team of chefs*, each in the same kitchen:
- Each chef can work on a different dish at the same time: one is chopping, another is boiling, another is baking.

This is **parallelism**: multiple tasks truly happening at the same time, speeding up the meal preparation.

# 2. Concurrency in Golang
## - Goroutines:
At the *heart of Go’s* approach to concurrency is the **goroutine**. Goroutines are lightweight, independently executing functions managed by the Go runtime (not the OS), making them much more efficient than traditional threads.
- **Lightweight**: Starting a goroutine costs very little memory (as little as a 2 KB stack, which grows as needed). You can easily run thousands—even millions—of goroutines in a single Go program.
- **Managed by Go Runtime**: The Go scheduler multiplexes goroutines onto a pool of operating system threads, so you don’t have to manage threads directly.

```go
go myFunction()
```

## - Channels:
Goroutines are powerful, but they’re even more useful when they work together. **Channels** are Go’s elegant way to **communicate and synchronize between goroutines**.
- Think of a channel as a pipe: one goroutine can send values in, and another can receive them out.
Channels are typed (chan int, chan string, etc.), ensuring safe and predictable communication.
They can also be buffered (holding a set number of values) or unbuffered (synchronizing sender and receiver).

```go
ch := make(chan int)    // Create a channel of int

go func() {
    ch <- 100            // Send value into channel
}()

val := <-ch             // Receive value from channel
fmt.Println(val)        // Output: 100
```

### Unbuffered Channels (make(chan T))
- An unbuffered channel has *no capacity*—it can hold zero elements.
- **Send blocks until receive**: When you send (ch <- val), the operation blocks until another goroutine is ready to receive (<-ch) from the channel.
- **Receive blocks until send**: When you receive from the channel, it blocks until a goroutine sends a value.
- **Use case**: Great for synchronization: ensures the sender and receiver rendezvous ("handshake").

### Buffered Channels (make(chan T, capacity))
- A buffered channel *has capacity* N—it can hold up to N elements.
- **Send blocks only if buffer is full**: The sender (ch <- val) only blocks when the buffer is full.
- **Receive blocks only if buffer is empty**: The receiver (<-ch) blocks only when the buffer is empty.
- **Use case**: Good for decoupling producer and consumer speeds; allows some "slack" between them.

Channels make it easy to coordinate work, fan out tasks, and collect results—without explicit locks or mutexes.

## - Example code:
You can see the code in the repository: https://github.com/LamThanhNguyen/Concurrency-Parallelism-in-Python-Golang

# 3. Concurrency in Python
## - Threads
Threads in Python allow you to write code that appears concurrent, with multiple threads doing work *‘at the same time’*.
- However, the Global Interpreter Lock (GIL) means that only one thread can execute Python bytecode at a time (even on multi-core CPUs). 
- This limits true parallelism for CPU-bound code, but threads are still useful for IO-bound tasks (like network calls or file operations).

## - Asyncio: Concurrency for IO-bound Work
**Asyncio** provides native support for asynchronous programming using async/await. 
This is ideal for managing thousands of simultaneous IO-bound operations, like HTTP requests or database calls, all in a single OS thread.
- Each fetch data coroutine runs asynchronously—when it hits await, control returns to the event loop, which can switch to another task.
- Ideal for network requests, APIs, or anything with lots of waiting.
- **Not for CPU-bound work**: Asyncio can’t run CPU-intensive code in parallel; it’s designed for IO-bound concurrency.

## - Example code:
You can see the code in the repository: https://github.com/LamThanhNguyen/Concurrency-Parallelism-in-Python-Golang

# 4. Parallelism in Golang
The environment variable **GOMAXPROCS** (or **runtime.GOMAXPROCS(n)** in code) sets the **maximum number of CPU cores** Go will use for running goroutines in parallel.

## Go Scheduler: The G-M-P Model
Go’s runtime uses a unique, high-performance scheduler, based on three concepts:
- **G** — Goroutine: the actual lightweight thread
- **M** — Machine: a wrapper for an OS thread
- **P** — Processor: a resource that schedules goroutines onto threads.
Only as many goroutines can run in parallel as there are Ps (set by GOMAXPROCS).

**Work Stealing:**
Each processor (P) has its own run queue of goroutines. If a P runs out of work, it “steals” goroutines from others, balancing the workload efficiently.

[Goroutine] -> [Processor] -> [Machine/OS thread] -> [CPU core]
- Many Gs are scheduled onto Ps.
- Ps are tied to Ms (OS threads).
- Ms are executed on real CPU cores.

This lets Go utilize all available CPU cores for compute-heavy workloads with minimal code changes.

## - Example code:
You can see the code in the repository: https://github.com/LamThanhNguyen/Concurrency-Parallelism-in-Python-Golang

# 5. Parallelism in Python
## - The GIL and Its Impact
Python’s most popular implementation (CPython) uses a mechanism called the Global Interpreter Lock (GIL). The GIL ensures that only one thread executes Python bytecode at a time—even on a multi-core machine.
- For CPU-bound code (math, data crunching, etc.), Python threads cannot run in true parallel. They take turns, so you get concurrency but not parallelism.
- For IO-bound code (network, disk, waiting), threads can be useful, because while one thread is waiting, another can work.

## - Multiprocessing for Real Parallelism
The multiprocessing module sidesteps the GIL by launching multiple processes, each with its own Python interpreter and memory space.
- Each process runs fully in parallel—one per CPU core—so you get true parallelism for heavy computations.
- Each process is heavier than a thread, as it has its own interpreter and memory space.

## - Example code:
You can see the code in the repository: https://github.com/LamThanhNguyen/Concurrency-Parallelism-in-Python-Golang

# 6. Conclustion
| Feature          | Go Goroutines          | Python Threading |	Python Asyncio    |	Python Multiprocessing |
| :--------------: | :--------------------: | :--------------: | :------------------: | :--------------------: |
| Lightweight      | ✔️ (very)             |  ❌ (heavy)      | ✔️ (light)           | ❌ (heavy)            |
| True parallelism | ✔️ (with GOMAXPROCS)  |  ❌ (GIL)        | ❌ (single-threaded) | ✔️ (separate process) |
| Best for         | IO & CPU-bound         | IO-bound         | IO-bound             |	CPU-bound              |


**Concurrency** and **parallelism** are key skills for any backend or systems engineer. **Golang’s goroutines and channels** make concurrent and parallel programming both easy and robust, while **Python** offers a range of tools for IO- and CPU-bound work—if you know which to use, and when. Try out the examples, check out the full code in my repository, and explore which model fits your next project best!