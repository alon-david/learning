### Embedded Recipes 2018 - Finding sources of latency in your system - Steven Rostedt [https://www.youtube.com/watch?v=iQCdVq85Coo&ab_channel=hupstream]

#### Hardware Latency

can't get better than what the hardware gives you

- sources of HW latency
  - System Management Interrupts (SMI)
    - The BIOS takes over the machine
  - Clock Frequency
    - the system slows down
  - Cache line bouncing
    - sharing the same variable among CPUs

`cyclictest` - a tool to measure latency

- runs a simple loop (in a high priority task)
  - sleep for specific time
  - get timestamp when wakes up
  - compare the difference
- best to use Nano sleep
  - can also use signals, but that has high latency
- Favorite application of the Linux RT folks