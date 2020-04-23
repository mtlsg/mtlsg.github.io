---
layout: post
title:  "BFD -- A brief overview"
date:   2020-04-18 11:17:45 -0400
author:	Jasminko Mulahusic
categories: networking
---
BFD (Bidirectional forwarding detection) is an IETF standard for rapid communication failure detection. Although there exist other mechanisms that can be used for the same purpose, BFD has some unique set of features that make it a very popular choice for use where fast failure detection is required.

The base BFD protocol specification is RFC5880. In addition to the base specification (RFC5880), there are a number of other companion standard documents that define more specific application use cases and specify how BFD can be used in combination with other underlaying transport technologies.

Some of the other BFD relateed RFCs are:
* RFC5881: [Bidirectional Forwarding Detection (BFD) for IPv4 and IPv6 (Single Hop)][rfc5880]
* RFC5882: [Generic Application of Bidirectional Forwarding Detection (BFD)][rfc5882]
* RFC5883: [Bidirectional Forwarding Detection (BFD) for Multihop Paths][rfc5883]
* RFC7130: [Bidirectional Forwarding Detection (BFD) on Link Aggregation Group (LAG) Interfaces)][rfc7130]
* RFC7419: [Common Interval Support in Bidirectional Forwarding Detection][rfc7419]
* RFC7880: [Seamless Bidirectional Forwarding Detection (S-BFD)][rfc7880]
* RFC8562: [Bidirectional Forwarding Detection (BFD) for Multipoint Networks][rfc8562]

### Failure detection vs network convergence

Sometimes, there is a rather incorrect belief that BFD is synonymous with network convergence. Although, BFD most often can and do result in faster network convergence, use of BFD in itself does not guarantee fast network convergence. The time it takes for a network to converge after a failure and find an alternative path between source and destination, depends on several factors in addition to how fast initial failure is detected. In other words, initial failure detection is just one step in the chain of events that need to happen before the network has converged. For instance, once connectivity failure has been detected between two routers, an alternative path has to be found (often with the help of a routing protocol) and forwarding engine has to be reprogrammed. In some cases, finding an alternative path can be a fast process and sometimes it can take longer time. For instance, if BFD is used with static routes, an alternative static route, hopefully with a reachable next hop, is usually already available and the alternative next hop can be installed instanly in the forwarding engine. For cases where dynamic routing protocols are used in conjunction with BFD, this process can take longer time depending if additional signalling by the routing protocols is required and how long the path computation process will take time.

In summary, BFD detection times does not necessarily directly translate to fast network convergence times.

### Other mechanisms to detect network failures

BFD is not the only tool available for rapid failure detection. Some data links (e.g. SONET) incorporate mechanisms that can be used for rapid failure detection. Most routing protocols also provide mechanisms, usually through periodic HELLO packets, that make it possible to detect failures, albeit not as fast or as efficient as what is possible with BFD. The point here is that depending on the overall network design goal with respect to the total target network convergence time, use of BFD may or may not be required. This is something that is usually addressed during the application requirements collection and  network design phases.

### BFD offload

One of the main advantages (and promises) of BFD is that, as opposed to routing protocols failure detection mechanisms, much of the CPU intensive BFD functionality can be offloaded to be executed in hardware instead of the control plane. This greatly increases feasibility to transmit regular hello packets at much faster intervals. Largely thanks to hardware offload, BFD is able to achieve much faster rapid detection as compared to other mechanisms that are entirely control based.

However, it is important to note that not all BFD implementations support hardware offload. Vendors usually do not state whether their BFD implementation supports hardware offload, leading customers to believe that hardware offload is used. When customers believe that BFD is implemented in hardware, such customers may be eager to use the most aggressive BFD timers available which might have negative impact on the control plane and produce negative side effects.

### Asynchronous vs Demand mode

BFD supports two main modes of operation: asynchronous and demand. Most implementations use asynchronous mode where a BFD session between two nodes is established and BFD hello packets are constantly exchanged between the two nodes on a predetermined interval, where absence of hello packets is interpreted as a network connectivity problem. Demand mode is rarer and once a BFD session has been established in demand mode, the nodes might decide to stop exchanging hello packets unless network connectivity verification is needed. Furthermore, demand mode requires that nodes have an independent way of verifying connectivity to the other node.

### Echo function

Echo function is another useful mode of BFD operation. When echo function is used, the source node will transmit a stream of echo packets which will be looped by the destination node. Both source and destination address of the echo packet will be the that of the node generating the echo packet stream. Since echo packets are simply looped in the forwarding engine, i.e. hardware, echo packets will not (should not) generate any additional load on the control plane. Loss of echo packets will indicate network connectivity issues prompting the generating node to signal the BFD session as down.

Use of echo function has generally following positive outcomes on BFD operation:
* Echo packets can be generated with shorter intervals, leading to faster failure detection.
* The rate of control plane hello packets (control hello packets are still needed for BFD session maintenance) can be decreased leading to less load on the control plane.

Echo function is used and enabled individually on each node and each node will signal whether they allow looping of echo packets or not.

> Note:
> 
> When echo function is used on a node, it is important to disable generation of ICMP unreachable packets on that interface. Recall that echo packets have same source/destination address and usually ICMP redirects would be generated for such cases which can lead in substantial load of ICMP packets on the link.

### Control plane independent flag

A node that implements and uses BFD can signal to its peer whether its BFD implementation is control plan independent or not. If a node sets the C bit in the BFD control packet, the node is signalling that its BFD implementation is decoupled for its control plan and is implemented in hardware, i.e. in the forwarding engine. That means that the node can continue to forward packets even if its control plane is not fully functional. This feature is particularly useful for various forms of GR and NSR. 

### BFD example

Here is a BFD example from a live system where BFD timers have been configured with different values for easier explanation.

Node 2 output:

{% highlight console %}
Session state is Up and using echo function with 300 ms interval
Local Diag: 0, Demand mode: 0, Poll bit: 0, Authentication: None
MinTxInt: 300000 us, MinRxInt: 2000000 us, Multiplier: 4
Received MinRxInt: 2000000 us, Received Multiplier: 3
Holdown (hits): 6000 ms (0), Hello (hits): 2000 ms (381674)
Rx Count: 468563, Rx Interval (ms) min/max/avg: 0/1596/1537 last: 978 ms ago
Tx Count: 381674, Tx Interval (ms) min/max/avg: 1887/1887/1887 last: 1864 ms ago
Registered protocols:  isis
Uptime: 8 days 8 hrs 7 mins 21 secs
Last packet: Version: 1                - Diagnostic: 0
             State bit: Up             - Demand bit: 0
             Poll bit: 0               - Final bit: 0
             Multiplier: 3             - Length: 24
             My Discr.: 1140850696     - Your Discr.: 1140850691
             Min tx interval: 500000   - Min rx interval: 2000000
             Min Echo interval: 50000  - Authentication bit: 0
Hosting LC: 1, Down reason: None, Reason not-hosted: None, Offloaded: No
{% endhighlight %}

From the output for node 2, the following can be concluded:
* Node 2 is using echo function.
* Asynchronous mode is used.
* Authentication is not used.
* Node 2 allows use of echo function with minimum interval of 50ms.
* From the output, it cannot be concluded if node 1 is using echo.
* Node 2 is advertising (and using) minimum transmit interval of 300ms.
* Node 2 is advertising minimum receive interval of 2000ms.
* Node 2 is using multiplier of 4.
* Node 1 is advertising minimum transmit interval of 500ms.
* Node 1 is advertising minimum receive interval of 2000ms.
* Node 1 is advertising multiplier of 3.
* Node 1 is allowing use of echo function with minimum interval of 50ms.
* This BFD session is running on LC1 on node 2 and is not running in hardware. (This particular vendor from where the output has been taken does not offload BFD sessions to hardware (forwarding engine) by default).

### Note about slow timers

Here is BFD configuration for nodes from the above example.

Node 1 BFD config:

{% highlight console %}
bfd slow-timer 2000
bfd echo-rx-interval 50

interface Ethernet1/1
  bfd interval 500 min_rx 600 multiplier 3
{% endhighlight %}

Node 2 BFD config:

{% highlight console %}
bfd slow-timer 2000
bfd echo-rx-interval 50

interface Ethernet1/1
  bfd interval 300 min_rx 400 multiplier 4
{% endhighlight %}

Why is node 2 advertising minimum receive interval of 2000ms even though it is configured with minimum receive interval of 400ms?

The reason for this is echo function in combination with a different configured `slow-timer` value. It has been mentioned earlier that when echo function has been enabled and used, periodic control plane hello packets necessary for BFD session maintenance can be slowed down. This is possible since echo function is now responsible to detect connectivity failure with the desired (and configured) detection interval.

So, what will be the new (and lower) periodic hello interval value?

This will be controlled by a timer called `slow-timer`. Whenever echo function is used, the node will advertise the `slow-timer` as the min receive interval, instead of the regular minimum receive `min_rx` interval.

[rfc5880]: https://tools.ietf.org/html/rfc5880
[rfc5881]: https://tools.ietf.org/html/rfc5881
[rfc5882]: https://tools.ietf.org/html/rfc5882
[rfc5883]: https://tools.ietf.org/html/rfc5883
[rfc7130]: https://tools.ietf.org/html/rfc7130
[rfc7419]: https://tools.ietf.org/html/rfc7419
[rfc7880]: https://tools.ietf.org/html/rfc7880
[rfc8562]: https://tools.ietf.org/html/rfc8562
