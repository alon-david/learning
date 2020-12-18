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

### Resiliency in Distributed Systems - GopherConSG 2018 [https://www.youtube.com/watch?v=iJX4wdf-Y3M&ab_channel=SingaporeGophers]

- Resiliency Patters

  - Timeouts - stop waiting for an answer
  - Retries - try again on failure
    - queue and retry wherever possible
    - Idempotency is important
  - Circuit Breakers - stop making calls to save systems
    - Hystrix
    - circumvent calls when system is unhealthy
  - Fallback - degrade gracefully
  - Rate-limit/throttling
  - bulk-heading
  - queuing
  - monitoring/alerting
  - canary releases
  - redundancies
  - Simian Army
    - chaos monkey
    - janitor monkey
    - conformity monkey
    - latency monkey

  Patterns are no _silver bullet_ 

  System *Fail*, deal with it -> Design your Systems for Failure

  "Release It! - Design and Deploy Production Ready Software" Micheal T. Nygard.

  

  ### High(er) Reliability Software Patterns for Go - GopherCon SG 2019 [https://www.youtube.com/watch?v=gB2dxBDjHP4&ab_channel=SingaporeGophers]
  
  https://github.com/getsentry/sentry - Real-time crash reporting for your web apps, mobile apps, and games.
  
  
  
  ### Golang UK Conference 2017 | Jack Lindamood - How to correctly use package context [https://www.youtube.com/watch?v=-_B5uQ4UGi0&list=WL&index=36&ab_channel=GopherConUK]
  
  - ```go
    httptrace.ClientTrace{}...
    
    ```
  
  - context.Value is used for immutable values
  
  - errgroup is a strong abstraction on top of context
  
  - context.Value should inform not control
  

### GopherCon UK 2018: Michal Witkowski - Abusing Go's Net Package for Fun and Profit [https://www.youtube.com/watch?v=6nkAIhz-8Nc&list=WL&index=28&ab_channel=GopherConUK]

```go
func DialContext(ctx context.Context, network, address string) (Conn, error)
```



the ability to return `net.Conn` of your choice and have access to `Context` is super useful.

```go
type MiddleWare func(http.Handler) http.Handler	
```



literally a wrapper for a handler. Recursively. Adopted by `chi` and `improbable-eng/go-httpwares`

Basically externalize common things:

- Logging
- Monitoring
- Tracing
- Debug pages (on `debug/request`)

```go
type TripperWare func(http.RoundTripper) http.RoundTripper
```

literally a wrapper for a round tripper. Recursively. What `chi` did for Handler we do for `RoundTripper`.

It's on the client side so it allows us to track outbound calls easily.

![image-20201214200003368](C:\Users\alond\learning\Go\[NOTES] golang lectures.assets\image-20201214200003368.png)

#### HTTP2 in short

- same semantics as `HTTP/1.1`
- multiplex request over conn
- Binary instead of text
- No head-of-line blocking
- Bidirectional by nature

![image-20201214200926958](C:\Users\alond\learning\Go\[NOTES] golang lectures.assets\image-20201214200926958.png)

github repos (in order of appearance)

- `mwitkow/go-http-dialer`
- `mwikow/go-conntrack`
- `go-chi/chi`
- `improbable-eng/go-httpwares`
- `improbable-eng/kedge`
- `grpc-ecosystem/go-grpc-middleware`
- `improbable/grpc-web`