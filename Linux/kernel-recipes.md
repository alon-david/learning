## Kernel Recipes

## YouTube

### Kernel Recipes 2015 - Kernel packet capture technologies - by Eric Leblond [https://www.youtube.com/watch?v=5gEWyCW-qx8]

- raw sockets - all users to write any packets

  - send low level message - icmp, igmp

  - implement new protocol in userspace

  - sniffing - 

    - capture traffic
    - promiscuous mode - get all traffic on interface
    - use by network monitoring tools
      -  tcpdump, wireshark

  - libpcap - multi os abstraction for packet capture

  - ```c
    #include <sys/socket.h>
    #include <netinet/in.h>
    raw_socket = socket(AF_INET, SOCK_RAW, int protocol);
    ```

    straight socket mode -

    - get packet per packet via `recvmsg`
    - optional `ioctl` - get timestamp

  - NAPI - 

    - internal path - 
      - data in card buffer
      - data copied to skb
      - data copied to socket
      - data read and copied by userspace

  - memory map 

    - internal path - 
      - data in card buffer
      - data copied to skb
      - data copied to ring buffer
      - userspace access data via pointer in ring buffer

  - AF_PACKET 

    - load balancing (2011)
      - kernel does load balancing
      - multiple algorithms
      - LB algorithm - 
        - round robin
        - flow: all packets of a given flow are send to the same socket
        - CPU: all packets treated in kernel by a CPU are send to the same socket.
      - RSS queues 
        - multi queue NIC have multiple TX RX
        - data can be split in multiple queues
          - programmed by user
          - flow load balanced
        - RSS queues load balancing
          - NIC does load balancing using hash function
          - CPU affinity is set to ensure we keep the cache line.

  - tpacket_v3 (2011)

    - the problem:
      - cells are fixed size
      - size is the one of biggest packet(MTU)
      - small packets use the same memory as big one
    - variable size cells
      - ring buffer
      - update memory mapping to enable variable sizes
      - use a get pointer to next cell approach

  - Netmap (2012)

    - similar approach than `PF_RING`
      - skip kernel path
      - put in ring buffer
    - user access the ring buffer
    - paired with network card ring 

  - `AF_PACKET` rollover option (2013)

    - single intensive flow
      - load balancing is flow based
      - one intensive flow saturate core capacity
      - load needs to be shared
    - ​	Principle
      - move to next ring when ring is full
      - as a load balancing mode
      - as a fallback method

  - DPKP (2012-)

    - set of libraries and driver
    - design for past packet processing
    - Architecture - 
      - multicore framework
      - huge page memory
      - ring buffers
      - poll mode drivers

    

  

### Metrics are moneyKernel Recipes 2019 [https://www.youtube.com/watch?v=SS9LBlXL5Xg&feature=emb_title&ab_channel=hupstream]



### Kernel Recipes 2018 - Packets probes and eBPF filtering in Skydive - Nicolas Planel [https://www.youtube.com/watch?v=cgtYIb2907g&ab_channel=hupstream]

#### what is skydive?

real-time network topology and protocol analyzer

why: Network is complex (SDN)

- Topologies (physical, overlay, application) are dynamic
- multiple opaque tunneling layers
- infrastructure complexity

Use cases

- Network topology visualization (Discovery)
- flow troubleshooting (capture)
- flow monitoring (grafana)
- workflow (validation, CI/CD)

|                       | Kernel | Overhead |                         Limitations                          |
| :-------------------: | :----: | :------: | :----------------------------------------------------------: |
|       AF_PACKET       |  all   |  Hugh!   |                             None                             |
| BPF (Syn/Fin/Rst/Ack) |  2.5+  |   Low    | No packets metrics. <br />TCP only.<br /> start and end sessions |
|         eBPF          | 3.18+  |   Low    | No Classification.<br />No fragmentation.<br />No tunneling  |

### Kernel Recipes 2019 - BPF at Facebook [https://www.youtube.com/watch?v=bbHFg9IsTk8&ab_channel=hupstream]

