# Go Language

## YouTube

### GopherCon 2019: Dave Cheney - Two Go Programs, Three Different Profiling Techniques [https://www.youtube.com/watch?v=nok0aYiGiYA&ab_channel=GopherAcademy]

- ``` go 
  - defer profile.Start(Profile.CPUProfile, profile.ProfilePath(".")).Stop()
  - defer profile.Start(Profile.MemProfile, profile.MemProfileRate(1), profile.ProfilePath(".")).Stop()
  - defer profile.Start(Profile.TraceProfile, profile.ProfilePath(".")).Stop()
  ```

- `go tool pprof cpu.pprof -http:=8080`

- `go tool trace trace.out`

- when a thread does not return from a syscall fast enough, go runtime creates another thread t o replace it and now we are extra thread that we need to sched - timeout is something like 20ms

### Escape Analysis and Memory Profiling - GopherCon SG 2017 [https://www.youtube.com/watch?v=2557w0qsDV0&ab_channel=SingaporeGophers]

- when sharing mem between functions it is allocated on the heap

- ```shell
  go build -gcflags "-m -m"
  ```

### Understanding Allocations: the Stack and the Heap - GopherCon SG 2019 [https://www.youtube.com/watch?v=ZMZpH4yT7M0&ab_channel=SingaporeGophers]

- sharing down _typically_ stays on the stack (passing pointers)

- sharing up _typically_ escapes to the heap (returning pointers)

- when possible, the go compilers will allocate variables that are local to function in that function's stack frame https://golang.org/doc/faq#stack_or_heap

- ```shell
  go help build
  go tool compile -h
  # "-m" print optimization decisions
  ```

- when are values constructed on the heap?

  1. when a value could possibly be refenced after the function that constructed the value returns.
  2. when the compiler determines a value is too large to fit on the stack
  3. when the compiler doesn't know the size of a value at compiler time

- some commonly allocated values

  - values shared with pointers
  - variables stored in interface variables 
  - func literal variables 
    - variables captured by a closure
  - backing data for maps channels slices and strings

- Points to remember

  - optimize for correctness, not performance
  - go only puts function variables on the stack if it can prove a variable is not used after the function returns
  - don't guess, use the tooling
    - pprof
    - tracer

### Garbage Collection Semantics - GopherCon SG 2019 [https://www.youtube.com/watch?v=q4HoWwdZUHs&ab_channel=SingaporeGophers]

- ```shell
  GODEBUG=gotrace=1כ
  ```

- misconception is thinking that slowing down the pace of the collector is a way to improve performance

- Don't do this

  - you could decide to change the GC percentage value to something larger than 100
  - this will increase the amount of heap memory that can be allocated before the next collection starts
  - this could result in the pace of collection to slow down, don't consider doing this

- Reduce Allocations

  - get more work don't between or during the collection
  - reduce the amount or the number of allocations any piece of work is adding to heap memory

- Two latency costs

  - stealing CPU capacity
  - Stop The World (STW) latency

- Reducing Stress on Heap Memory

  - stress can be defined as how fast the application is allocating heap memory within a given amount of time
  - when stress is reduced, the latencies being inflicted by the collector will be reduced
  - It's the GC latencies that are slowing down you application.

