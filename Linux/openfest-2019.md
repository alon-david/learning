### Steven Rostedt - Learning the Linux Kernel with tracing [https://www.youtube.com/watch?v=JRyrhsx-L5Y&ab_channel=OpenfestBulgaria]

- `strace`
  - see all the interface into the kernel
  - traces all "system calls"
  - uses "ptrace" 
    - same as gdb
  - strace used to slow applications by 30%, after "spectre" and "meltdown" were discovered the slowdown shrink to 8%.

#### The linux kernel internal tracer - `ftrace`

- the official tracer of the linux kernel
- probaly already enabled in the linux kernel you are running
- `/sys/kernel/tracing`
- `/sys/kernel/debug/tracing`
- only needs `echo` and `cat`

`trace-cmd`

- a command line tool that does the work for you
- run as root
- will mount `/sys/kernel/tracing`
- can save the trace offline
- down from `git://git.kernel.org/pub/scm/utils/trace-cmd/trace-cmd.git`
- can record a lot of information
- some times, too much information is worse than too little
- it takes an expert to be able to decipher what is happening

`KernelShark`

- can read `trace.dat` files
- it gives a visualization of what is happening
- much easier to parse large amount of data



