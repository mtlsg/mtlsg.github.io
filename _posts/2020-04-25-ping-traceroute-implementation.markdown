---
layout: post
title:  "ping and traceroute implementation"
date:   2020-04-25 10:34:09 -0400
author:	Jasminko Mulahusic
categories: networking
---
Ping and traceroute are two essential tools for troubleshooting network connectivity issues. These two tools will most probably be the starting point when trying to diagnose the root cause of a connectivity problem. Even though ping and traceroute are mostly used by network and system administrators, it is not uncommon for non-technical computer users to having to collect output of ping and traceroute as a step of a troubleshooting procedure.

Having understanding of how ping and traceroute work and which techniques and protocols they are based upon is very important for a successful outcome when troubleshooting. ping and traceroute can often indicate what might be the cause of a connectivity issue, where in the network the issue may be happening (at which hop), whether there really is a issue to begin in the first place (ping and/or traceroute may be failing, but application traffic may be just fine) and finally what can be done to fix the problem.

The purpose of this post is not to go into great details about how ping and traceroute work in the first place, but to highlight how different operating systems implement ping and traceroute applications. I.e. this boils down to the transport protocol over which ping and traceroute operate.

> This is in no way an extensive list of all possible flavors of different operating systems and implementations. The operating systems listed here are the the most common ones.

Why is it important to be familiar how ping and traceroute are implemented and what default values they use in different operating systems? Because having knowledge about the particulars of the implementation can help in understanding the likely root cause of a network connectivity problem. Here are some examples that are not uncommon when troubleshooting connectiviy and application issues:

* Ping towards the destination is working, but traceroute is failing.
* Traceroute towards the destination is working, but ping is failing.
* Ping reports packet loss, while traceroute does not and vice versa.
* Etc.

### Ping

Ping is implemented by source node sending an ICMP ECHO REQUEST packet (type 8, code 0) to the destination node and destination node responding by ICMP ECHO REPLY packet (type 0, code 0).

#### Windows

Below is ping packet capture from a Windows 10 host:

{% highlight console %}
16:18:43.316443 IP (tos 0x0, ttl 128, id 49659, offset 0, flags [none], proto ICMP (1), length 60)
    X.X.X.X > Y.Y.Y.Y: ICMP echo request, id 1, seq 577, length 40
16:18:43.316486 IP (tos 0x0, ttl 64, id 60029, offset 0, flags [none], proto ICMP (1), length 60)
    Y.Y.Y.Y > X.X.X.X: ICMP echo reply, id 1, seq 577, length 40
16:18:44.343637 IP (tos 0x0, ttl 128, id 49661, offset 0, flags [none], proto ICMP (1), length 60)
    X.X.X.X > Y.Y.Y.Y: ICMP echo request, id 1, seq 578, length 40
16:18:44.343680 IP (tos 0x0, ttl 64, id 60078, offset 0, flags [none], proto ICMP (1), length 60)
    Y.Y.Y.Y > X.X.X.X: ICMP echo reply, id 1, seq 578, length 40
{% endhighlight %}

As can be seen from the above, Windows 10 default ping command uses ICMP ECHO request/reply with TTL set to 128. Windows 10 provides various options available that can be used to alter default ping values.

#### Linux

Below is ping packet capture from a Linux host:

{% highlight console %}
16:33:55.576998 IP (tos 0x0, ttl 64, id 35062, offset 0, flags [DF], proto ICMP (1), length 84)
    X.X.X.X > Y.Y.Y.Y: ICMP echo request, id 19209, seq 1, length 64
16:33:55.578135 IP (tos 0x0, ttl 127, id 11510, offset 0, flags [none], proto ICMP (1), length 84)
    Y.Y.Y.Y > X.X.X.X: ICMP echo reply, id 19209, seq 1, length 64
16:33:56.578534 IP (tos 0x0, ttl 64, id 35118, offset 0, flags [DF], proto ICMP (1), length 84)
    X.X.X.X > Y.Y.Y.Y: ICMP echo request, id 19209, seq 2, length 64
16:33:56.579094 IP (tos 0x0, ttl 127, id 11511, offset 0, flags [none], proto ICMP (1), length 84)
    Y.Y.Y.Y > X.X.X.X: ICMP echo reply, id 19209, seq 2, length 64
{% endhighlight %}

As can be seen from the above, Linux default ping command uses ICMP ECHO request/reply with TTL set to 64. Note that Linux host will by default set the DF flag. Linux provides various options available that can be used to alter the default ping values.

#### Cisco IOS

Below is ping packet capture from a Cisco IOS host:

{% highlight console %}
15:04:01.623897 IP (tos 0x0, ttl 255, id 20, offset 0, flags [none], proto ICMP (1), length 100)
    X.X.X.X > Y.Y.Y.Y: ICMP echo request, id 4, seq 0, length 80
15:04:01.623962 IP (tos 0x0, ttl 64, id 36373, offset 0, flags [none], proto ICMP (1), length 100)
    Y.Y.Y.Y > X.X.X.X: ICMP echo reply, id 4, seq 0, length 80
15:04:01.625767 IP (tos 0x0, ttl 255, id 21, offset 0, flags [none], proto ICMP (1), length 100)
    X.X.X.X > Y.Y.Y.Y: ICMP echo request, id 4, seq 1, length 80
15:04:01.625797 IP (tos 0x0, ttl 64, id 36374, offset 0, flags [none], proto ICMP (1), length 100)
    Y.Y.Y.Y > X.X.X.X: ICMP echo reply, id 4, seq 1, length 80
{% endhighlight %}

As can be seen from the above, Cisco IOS default ping command uses ICMP ECHO request/reply with TTL set to 255. Cisco IOS provides various options available that can be used to alter the default ping values.

#### Cisco IOS-XR

Below is ping packet capture from a Cisco IOS-XR host:

{% highlight console %}
16:05:50.344458 IP (tos 0x0, ttl 255, id 0, offset 0, flags [none], proto ICMP (1), length 100)
    X.X.X.X > Y.Y.Y.Y: ICMP echo request, id 49308, seq 0, length 80
16:05:50.357676 IP (tos 0x0, ttl 54, id 0, offset 0, flags [none], proto ICMP (1), length 96)
    Y.Y.Y.Y > X.X.X.X: ICMP echo reply, id 49308, seq 0, length 76
16:05:50.359646 IP (tos 0x0, ttl 255, id 1, offset 0, flags [none], proto ICMP (1), length 100)
    X.X.X.X > Y.Y.Y.Y: ICMP echo request, id 49308, seq 1, length 80
16:05:50.372862 IP (tos 0x0, ttl 54, id 0, offset 0, flags [none], proto ICMP (1), length 96)
    Y.Y.Y.Y > X.X.X.X: ICMP echo reply, id 49308, seq 1, length 76
{% endhighlight %}

As can be seen from the above, Cisco IOS-XR default ping command uses ICMP ECHO request/reply with TTL set to 255. Cisco IOS-XR provides various options available that can be used to alter the default ping values.

#### Cisco IOS-XE

Below is ping packet capture from a Cisco IOS-XE host:

{% highlight console %}
16:08:38.494979 IP (tos 0x0, ttl 255, id 7, offset 0, flags [none], proto ICMP (1), length 100)
    X.X.X.X > Y.Y.Y.Y: ICMP echo request, id 2, seq 0, length 80
16:08:38.508194 IP (tos 0x0, ttl 54, id 0, offset 0, flags [none], proto ICMP (1), length 96)
    Y.Y.Y.Y > X.X.X.X: ICMP echo reply, id 2, seq 0, length 76
16:08:38.510434 IP (tos 0x0, ttl 255, id 8, offset 0, flags [none], proto ICMP (1), length 100)
    X.X.X.X > Y.Y.Y.Y: ICMP echo request, id 2, seq 1, length 80
16:08:38.523639 IP (tos 0x0, ttl 54, id 0, offset 0, flags [none], proto ICMP (1), length 96)
    Y.Y.Y.Y > X.X.X.X: ICMP echo reply, id 2, seq 1, length 76
{% endhighlight %}

As can be seen from the above, Cisco IOS-XE default ping command uses ICMP ECHO request/reply with TTL set to 255. Cisco IOS-XE provides various options available that can be used to alter the default ping values.

#### Juniper JunOS

Below is ping packet capture from a Juniper JunOS host:

{% highlight console %}
10:58:27.626516 IP (tos 0x0, ttl 64, id 11341, offset 0, flags [none], proto ICMP (1), length 84)
    X.X.X.X > Y.Y.Y.Y: ICMP echo request, id 64380, seq 0, length 64
10:58:27.626556 IP (tos 0x0, ttl 64, id 54166, offset 0, flags [none], proto ICMP (1), length 84)
    Y.Y.Y.Y > X.X.X.X: ICMP echo reply, id 64380, seq 0, length 64
10:58:28.628200 IP (tos 0x0, ttl 64, id 11342, offset 0, flags [none], proto ICMP (1), length 84)
    X.X.X.X > Y.Y.Y.Y: ICMP echo request, id 64380, seq 1, length 64
10:58:28.628250 IP (tos 0x0, ttl 64, id 54346, offset 0, flags [none], proto ICMP (1), length 84)
    Y.Y.Y.Y > X.X.X.X: ICMP echo reply, id 64380, seq 1, length 64 
{% endhighlight %}

As can be seen from the above, Juniper JunOS default ping command uses ICMP ECHO request/reply with TTL set to 64. Juniper JunOS provides various options available that can be used to alter the default ping values.

#### Nokia TiMOS

Below is ping packet capture from a Nokia TiMOS host:

{% highlight console %}
16:21:21.035167 IP (tos 0x0, ttl 64, id 6, offset 0, flags [none], proto ICMP (1), length 84)
    X.X.X.X > Y.Y.Y.Y: ICMP echo request, id 49156, seq 1, length 64
16:21:21.035217 IP (tos 0x0, ttl 64, id 37779, offset 0, flags [none], proto ICMP (1), length 84)
    Y.Y.Y.Y > X.X.X.X: ICMP echo reply, id 49156, seq 1, length 64
16:21:22.036071 IP (tos 0x0, ttl 64, id 7, offset 0, flags [none], proto ICMP (1), length 84)
    X.X.X.X > Y.Y.Y.Y: ICMP echo request, id 49156, seq 2, length 64
16:21:22.036126 IP (tos 0x0, ttl 64, id 37839, offset 0, flags [none], proto ICMP (1), length 84)
    Y.Y.Y.Y > X.X.X.X: ICMP echo reply, id 49156, seq 2, length 64 
{% endhighlight %}

As it can be seen from the above, Nokia TiMOS default ping command uses ICMP ECHO request/reply with TTL set to 64. Nokia TiMOS provides various options available that can be used to alter the default ping values.

### Traceroute

As opposed to ping implementation, where it has been observed that ICMP ECHO is used exclusively by all implementations, traceroute implementations vary between using ICMP and UDP as the main probing protocol.

#### Windows

Below is traceroute packet capture from a Windows 10 host:

{% highlight console %}
15:18:52.732322 IP (tos 0x0, ttl 1, id 52656, offset 0, flags [none], proto ICMP (1), length 92)
    X.X.X.X > Y.Y.Y.Y: ICMP echo request, id 1, seq 28, length 72
15:18:52.732497 IP (tos 0x0, ttl 64, id 48936, offset 0, flags [none], proto ICMP (1), length 56)
    A.A.A.A > X.X.X.X: ICMP time exceeded in-transit, length 36
        IP (tos 0x0, ttl 1, id 52656, offset 0, flags [none], proto ICMP (1), length 92)
    X.X.X.X > Y.Y.Y.Y: ICMP echo request, id 1, seq 28, length 72
15:18:52.733600 IP (tos 0x0, ttl 1, id 52657, offset 0, flags [none], proto ICMP (1), length 92)
    X.X.X.X > Y.Y.Y.Y: ICMP echo request, id 1, seq 29, length 72
15:18:52.733629 IP (tos 0x0, ttl 64, id 38678, offset 0, flags [none], proto ICMP (1), length 56)
    A.A.A.A > X.X.X.X: ICMP time exceeded in-transit, length 36
        IP (tos 0x0, ttl 1, id 52657, offset 0, flags [none], proto ICMP (1), length 92)
    X.X.X.X > Y.Y.Y.Y: ICMP echo request, id 1, seq 29, length 72
15:18:52.735079 IP (tos 0x0, ttl 1, id 52658, offset 0, flags [none], proto ICMP (1), length 92)
    X.X.X.X > Y.Y.Y.Y: ICMP echo request, id 1, seq 30, length 72
15:18:52.735106 IP (tos 0x0, ttl 64, id 44275, offset 0, flags [none], proto ICMP (1), length 56)
    A.A.A.A > X.X.X.X: ICMP time exceeded in-transit, length 36
        IP (tos 0x0, ttl 1, id 52658, offset 0, flags [none], proto ICMP (1), length 92)
    X.X.X.X > Y.Y.Y.Y: ICMP echo request, id 1, seq 30, length 72

15:18:53.750094 IP (tos 0x0, ttl 2, id 52659, offset 0, flags [none], proto ICMP (1), length 92)
    X.X.X.X > Y.Y.Y.Y: ICMP echo request, id 1, seq 31, length 72
15:18:53.759626 IP (tos 0xc0, ttl 63, id 7934, offset 0, flags [none], proto ICMP (1), length 120)
    B.B.B.B > X.X.X.X: ICMP time exceeded in-transit, length 100
        IP (tos 0x0, ttl 1, id 52659, offset 0, flags [none], proto ICMP (1), length 92)
    X.X.X.X > Y.Y.Y.Y: ICMP echo request, id 1, seq 31, length 72
15:18:53.760863 IP (tos 0x0, ttl 2, id 52660, offset 0, flags [none], proto ICMP (1), length 92)
    X.X.X.X > Y.Y.Y.Y: ICMP echo request, id 1, seq 32, length 72
15:18:53.771165 IP (tos 0xc0, ttl 63, id 7935, offset 0, flags [none], proto ICMP (1), length 120)
    B.B.B.B > X.X.X.X: ICMP time exceeded in-transit, length 100
        IP (tos 0x0, ttl 1, id 52660, offset 0, flags [none], proto ICMP (1), length 92)
    X.X.X.X > Y.Y.Y.Y: ICMP echo request, id 1, seq 32, length 72
15:18:53.779999 IP (tos 0x0, ttl 2, id 52661, offset 0, flags [none], proto ICMP (1), length 92)
    X.X.X.X > Y.Y.Y.Y: ICMP echo request, id 1, seq 33, length 72
15:18:53.788143 IP (tos 0xc0, ttl 63, id 7936, offset 0, flags [none], proto ICMP (1), length 120)
    B.B.B.B > X.X.X.X: ICMP time exceeded in-transit, length 100
        IP (tos 0x0, ttl 1, id 52661, offset 0, flags [none], proto ICMP (1), length 92)
    X.X.X.X > Y.Y.Y.Y: ICMP echo request, id 1, seq 33, length 72
{% endhighlight %}

As can be seen from the above, Windows 10 default traceroute command uses ICMP ECHO request/reply with TTL initially set to 1 and increased by 1 for every third probe. Windows 10 provides few options available that can be used to alter default traceroute values.

> Windows is one of the few operating systems that uses ICMP as the transport protocol for traceroute application.

#### Linux

Below is traceroute packet capture from a Linux host:

{% highlight console %}
09:36:56.228690 IP (tos 0x0, ttl 1, id 57706, offset 0, flags [none], proto UDP (17), length 60)
    X.X.X.X.36464 > Y.Y.Y.Y.33434: [bad udp cksum 0x1a8b -> 0xdf3c!] UDP, length 32
09:36:56.228738 IP (tos 0xc0, ttl 64, id 62311, offset 0, flags [none], proto ICMP (1), length 88)
    A.A.A.A > X.X.X.X: ICMP time exceeded in-transit, length 68
        IP (tos 0x0, ttl 1, id 57706, offset 0, flags [none], proto UDP (17), length 60)
    X.X.X.X.36464 > Y.Y.Y.Y.33434: [bad udp cksum 0x1a8b -> 0xdf3c!] UDP, length 32
09:36:56.228802 IP (tos 0x0, ttl 1, id 57707, offset 0, flags [none], proto UDP (17), length 60)
    X.X.X.X.49861 > Y.Y.Y.Y.33435: [bad udp cksum 0x1a8b -> 0xaae6!] UDP, length 32
09:36:56.228825 IP (tos 0xc0, ttl 64, id 62312, offset 0, flags [none], proto ICMP (1), length 88)
    A.A.A.A > X.X.X.X: ICMP time exceeded in-transit, length 68
        IP (tos 0x0, ttl 1, id 57707, offset 0, flags [none], proto UDP (17), length 60)
    X.X.X.X.49861 > Y.Y.Y.Y.33435: [bad udp cksum 0x1a8b -> 0xaae6!] UDP, length 32
09:36:56.228843 IP (tos 0x0, ttl 1, id 57708, offset 0, flags [none], proto UDP (17), length 60)
    X.X.X.X.39790 > Y.Y.Y.Y.33436: [bad udp cksum 0x1a8b -> 0xd23c!] UDP, length 32
09:36:56.228863 IP (tos 0xc0, ttl 64, id 62313, offset 0, flags [none], proto ICMP (1), length 88)
    A.A.A.A > X.X.X.X: ICMP time exceeded in-transit, length 68
        IP (tos 0x0, ttl 1, id 57708, offset 0, flags [none], proto UDP (17), length 60)
    X.X.X.X.39790 > Y.Y.Y.Y.33436: [bad udp cksum 0x1a8b -> 0xd23c!] UDP, length 32
09:36:56.228880 IP (tos 0x0, ttl 2, id 57709, offset 0, flags [none], proto UDP (17), length 60)
    X.X.X.X.56354 > Y.Y.Y.Y.33437: [bad udp cksum 0x1a8b -> 0x9187!] UDP, length 32
09:36:56.229003 IP (tos 0x0, ttl 2, id 57710, offset 0, flags [none], proto UDP (17), length 60)
    X.X.X.X.44854 > Y.Y.Y.Y.33438: [bad udp cksum 0x1a8b -> 0xbe72!] UDP, length 32
09:36:56.229128 IP (tos 0x0, ttl 2, id 57711, offset 0, flags [none], proto UDP (17), length 60)
    X.X.X.X.57743 > Y.Y.Y.Y.33439: [bad udp cksum 0x1a8b -> 0x8c18!] UDP, length 32
09:36:56.229162 IP (tos 0x0, ttl 3, id 57712, offset 0, flags [none], proto UDP (17), length 60)
    X.X.X.X.59475 > Y.Y.Y.Y.33440: [bad udp cksum 0x1a8b -> 0x8553!] UDP, length 32
09:36:56.229227 IP (tos 0x0, ttl 3, id 57713, offset 0, flags [none], proto UDP (17), length 60)
    X.X.X.X.48861 > Y.Y.Y.Y.33441: [bad udp cksum 0x1a8b -> 0xaec8!] UDP, length 32
09:36:56.229285 IP (tos 0x0, ttl 3, id 57714, offset 0, flags [none], proto UDP (17), length 60)
    X.X.X.X.57301 > Y.Y.Y.Y.33442: [bad udp cksum 0x1a8b -> 0x8dcf!] UDP, length 32
09:36:56.229366 IP (tos 0x0, ttl 4, id 57715, offset 0, flags [none], proto UDP (17), length 60)
    X.X.X.X.46984 > Y.Y.Y.Y.33443: [bad udp cksum 0x1a8b -> 0xb61b!] UDP, length 32
09:36:56.229414 IP (tos 0xc0, ttl 62, id 9804, offset 0, flags [none], proto ICMP (1), length 88)
    C.C.C.C > X.X.X.X: ICMP time exceeded in-transit, length 68
        IP (tos 0x0, ttl 1, id 57712, offset 0, flags [none], proto UDP (17), length 60)
    X.X.X.X.59475 > Y.Y.Y.Y.33440: [udp sum ok] UDP, length 32
09:36:56.229439 IP (tos 0x0, ttl 4, id 57716, offset 0, flags [none], proto UDP (17), length 60)
    X.X.X.X.39385 > Y.Y.Y.Y.33444: [bad udp cksum 0x1a8b -> 0xd3c9!] UDP, length 32
09:36:56.229479 IP (tos 0x0, ttl 4, id 57717, offset 0, flags [none], proto UDP (17), length 60)
    X.X.X.X.41795 > Y.Y.Y.Y.33445: [bad udp cksum 0x1a8b -> 0xca5e!] UDP, length 32
09:36:56.229516 IP (tos 0x0, ttl 5, id 57718, offset 0, flags [none], proto UDP (17), length 60)
    X.X.X.X.55823 > Y.Y.Y.Y.33446: [bad udp cksum 0x1a8b -> 0x9391!] UDP, length 32
09:36:56.229550 IP (tos 0x0, ttl 5, id 57719, offset 0, flags [none], proto UDP (17), length 60)
    X.X.X.X.54830 > Y.Y.Y.Y.33447: [bad udp cksum 0x1a8b -> 0x9771!] UDP, length 32
09:36:56.229589 IP (tos 0x0, ttl 5, id 57720, offset 0, flags [none], proto UDP (17), length 60)
    X.X.X.X.59269 > Y.Y.Y.Y.33448: [bad udp cksum 0x1a8b -> 0x8619!] UDP, length 32
09:36:56.229662 IP (tos 0xc0, ttl 62, id 3987, offset 0, flags [none], proto ICMP (1), length 88)
    C.C.C.C > X.X.X.X: ICMP time exceeded in-transit, length 68
        IP (tos 0x0, ttl 1, id 57713, offset 0, flags [none], proto UDP (17), length 60)
    X.X.X.X.48861 > Y.Y.Y.Y.33441: [udp sum ok] UDP, length 32
09:36:56.229678 IP (tos 0xc0, ttl 62, id 3988, offset 0, flags [none], proto ICMP (1), length 88)
    C.C.C.C > X.X.X.X: ICMP time exceeded in-transit, length 68
        IP (tos 0x0, ttl 1, id 57714, offset 0, flags [none], proto UDP (17), length 60)
09:36:56.229711 IP (tos 0x0, ttl 6, id 57721, offset 0, flags [none], proto UDP (17), length 60)
    X.X.X.X.38528 > Y.Y.Y.Y.33449: [bad udp cksum 0x1a8b -> 0xd71d!] UDP, length 32
09:36:56.229918 IP (tos 0x0, ttl 254, id 33054, offset 0, flags [none], proto ICMP (1), length 96)
    B.B.B.B > X.X.X.X: ICMP time exceeded in-transit, length 76
        IP (tos 0x0, ttl 1, id 57709, offset 0, flags [none], proto UDP (17), length 60)
    X.X.X.X.56354 > Y.Y.Y.Y.33437: [udp sum ok] UDP, length 32
09:36:56.229935 IP (tos 0x0, ttl 254, id 33055, offset 0, flags [none], proto ICMP (1), length 96)
    B.B.B.B > X.X.X.X: ICMP time exceeded in-transit, length 76
        IP (tos 0x0, ttl 1, id 57710, offset 0, flags [none], proto UDP (17), length 60)
    X.X.X.X.44854 > Y.Y.Y.Y.33438: [udp sum ok] UDP, length 32
09:36:56.229944 IP (tos 0x0, ttl 254, id 14845, offset 0, flags [none], proto ICMP (1), length 96)
    B.B.B.B > X.X.X.X: ICMP time exceeded in-transit, length 76
        IP (tos 0x0, ttl 1, id 57711, offset 0, flags [none], proto UDP (17), length 60)
    X.X.X.X.57743 > Y.Y.Y.Y.33439: [udp sum ok] UDP, length 32
{% endhighlight %}

As it can be seen from the above, Linux default traceroute command uses UDP packets with TTL initially set to 1 and increased by 1 for every third probe. Linux provides few options available that can be used to alter default traceroute values.

The Linux output above shows some other interesting things:
* Multiple probes are transmitted in parallel with different TTL values.
* ICMP unreachable messages do not have to be received in order reflecting the actual hops and can still be properly mapped to the proper hop that generated the response. How is that done? The clue is in the ICMP response packet.

#### Cisco IOS

Below is traceroute packet capture from a Cisco IOS host:

{% highlight console %}
10:44:06.757563 IP (tos 0x0, ttl 1, id 65, offset 0, flags [none], proto UDP (17), length 28)                                                      
    X.X.X.X.40567 > Y.Y.Y.Y.33434: [udp sum ok] UDP, length 0
10:44:06.757606 IP (tos 0xc0, ttl 64, id 59179, offset 0, flags [none], proto ICMP (1), length 56)
    A.A.A.A > X.X.X.X: ICMP time exceeded in-transit, length 36
        IP (tos 0x0, ttl 1, id 65, offset 0, flags [none], proto UDP (17), length 28)
    X.X.X.X.40567 > Y.Y.Y.Y.33434: [udp sum ok] UDP, length 0
10:44:15.769940 IP (tos 0x0, ttl 1, id 69, offset 0, flags [none], proto UDP (17), length 28)
    X.X.X.X.33625 > Y.Y.Y.Y.33435: [udp sum ok] UDP, length 0
10:44:15.769995 IP (tos 0xc0, ttl 64, id 59204, offset 0, flags [none], proto ICMP (1), length 56)
    A.A.A.A > X.X.X.X: ICMP time exceeded in-transit, length 36
        IP (tos 0x0, ttl 1, id 69, offset 0, flags [none], proto UDP (17), length 28)
    X.X.X.X.33625 > Y.Y.Y.Y.33435: [udp sum ok] UDP, length 0
10:44:15.771541 IP (tos 0x0, ttl 1, id 70, offset 0, flags [none], proto UDP (17), length 28)
    X.X.X.X.37740 > Y.Y.Y.Y.33436: [udp sum ok] UDP, length 0
10:44:15.771564 IP (tos 0xc0, ttl 64, id 59205, offset 0, flags [none], proto ICMP (1), length 56)
    A.A.A.A > X.X.X.X: ICMP time exceeded in-transit, length 36
        IP (tos 0x0, ttl 1, id 70, offset 0, flags [none], proto UDP (17), length 28)
    X.X.X.X.37740 > Y.Y.Y.Y.33436: [udp sum ok] UDP, length 0

10:44:15.773056 IP (tos 0x0, ttl 2, id 71, offset 0, flags [none], proto UDP (17), length 28)
    X.X.X.X.41879 > Y.Y.Y.Y.33437: [udp sum ok] UDP, length 0
10:44:15.773913 IP (tos 0x0, ttl 254, id 64331, offset 0, flags [none], proto ICMP (1), length 96)
    B.B.B.B > X.X.X.X: ICMP time exceeded in-transit, length 76
        IP (tos 0x0, ttl 1, id 71, offset 0, flags [none], proto UDP (17), length 28)
    X.X.X.X.41879 > Y.Y.Y.Y.33437: [udp sum ok] UDP, length 0
10:44:24.787511 IP (tos 0x0, ttl 2, id 75, offset 0, flags [none], proto UDP (17), length 28)
    X.X.X.X.42213 > Y.Y.Y.Y.33438: [udp sum ok] UDP, length 0
10:44:24.788640 IP (tos 0x0, ttl 254, id 23609, offset 0, flags [none], proto ICMP (1), length 96)
    B.B.B.B > X.X.X.X: ICMP time exceeded in-transit, length 76
        IP (tos 0x0, ttl 1, id 75, offset 0, flags [none], proto UDP (17), length 28)
    X.X.X.X.42213 > Y.Y.Y.Y.33438: [udp sum ok] UDP, length 0
10:44:24.790329 IP (tos 0x0, ttl 2, id 76, offset 0, flags [none], proto UDP (17), length 28)
    X.X.X.X.34332 > Y.Y.Y.Y.33439: [udp sum ok] UDP, length 0
10:44:24.791303 IP (tos 0x0, ttl 254, id 23610, offset 0, flags [none], proto ICMP (1), length 96)
    B.B.B.B > X.X.X.X: ICMP time exceeded in-transit, length 76
        IP (tos 0x0, ttl 1, id 76, offset 0, flags [none], proto UDP (17), length 28)
    X.X.X.X.34332 > Y.Y.Y.Y.33439: [udp sum ok] UDP, length 0

10:44:24.792839 IP (tos 0x0, ttl 3, id 77, offset 0, flags [none], proto UDP (17), length 28)
    X.X.X.X.32883 > Y.Y.Y.Y.33440: [udp sum ok] UDP, length 0
10:44:24.793269 IP (tos 0xc0, ttl 62, id 51592, offset 0, flags [none], proto ICMP (1), length 56)
    B.B.B.B > X.X.X.X: ICMP time exceeded in-transit, length 36
        IP (tos 0x0, ttl 1, id 77, offset 0, flags [none], proto UDP (17), length 28)
    X.X.X.X.32883 > Y.Y.Y.Y.33440: [udp sum ok] UDP, length 0
10:44:33.805200 IP (tos 0x0, ttl 3, id 81, offset 0, flags [none], proto UDP (17), length 28)
    X.X.X.X.36561 > Y.Y.Y.Y.33441: [udp sum ok] UDP, length 0
10:44:33.805673 IP (tos 0xc0, ttl 62, id 7208, offset 0, flags [none], proto ICMP (1), length 56)
    C.C.C.C > X.X.X.X: ICMP time exceeded in-transit, length 36
        IP (tos 0x0, ttl 1, id 81, offset 0, flags [none], proto UDP (17), length 28)
    X.X.X.X.36561 > Y.Y.Y.Y.33441: [udp sum ok] UDP, length 0
10:44:42.821107 IP (tos 0x0, ttl 3, id 85, offset 0, flags [none], proto UDP (17), length 28)
    X.X.X.X.39877 > Y.Y.Y.Y.33442: [udp sum ok] UDP, length 0
10:44:42.821561 IP (tos 0xc0, ttl 62, id 54005, offset 0, flags [none], proto ICMP (1), length 56)
    B.B.B.B > X.X.X.X: ICMP time exceeded in-transit, length 36
        IP (tos 0x0, ttl 1, id 85, offset 0, flags [none], proto UDP (17), length 28)
    X.X.X.X.39877 > Y.Y.Y.Y.33442: [udp sum ok] UDP, length 0
{% endhighlight %}

As it can be seen from the above, Cisco IOS default traceroute command uses UDP packets with TTL initially set to 1 and increased by 1 for every third probe. Cisco provides few options available that can be used to alter default traceroute values.

#### Cisco IOS-XR

Below is traceroute packet capture from a Cisco IOS-XR host:

{% highlight console %}
16:03:30.392001 IP (tos 0x0, ttl 1, id 51792, offset 0, flags [none], proto UDP (17), length 28)                                                     
    X.X.X.X.51792 > Y.Y.Y.Y.33435: [udp sum ok] UDP, length 0
16:03:30.392042 IP (tos 0xc0, ttl 64, id 56342, offset 0, flags [none], proto ICMP (1), length 56)
    A.A.A.A > X.X.X.X: ICMP time exceeded in-transit, length 36
        IP (tos 0x0, ttl 1, id 51792, offset 0, flags [none], proto UDP (17), length 28)
    X.X.X.X.51792 > Y.Y.Y.Y.33435: [udp sum ok] UDP, length 0
16:03:30.412110 IP (tos 0x0, ttl 1, id 51793, offset 0, flags [none], proto UDP (17), length 28)
    X.X.X.X.51792 > Y.Y.Y.Y.33436: [udp sum ok] UDP, length 0
16:03:30.412173 IP (tos 0xc0, ttl 64, id 56344, offset 0, flags [none], proto ICMP (1), length 56)
    A.A.A.A > X.X.X.X: ICMP time exceeded in-transit, length 36
        IP (tos 0x0, ttl 1, id 51793, offset 0, flags [none], proto UDP (17), length 28)
    X.X.X.X.51792 > Y.Y.Y.Y.33436: [udp sum ok] UDP, length 0
16:03:30.442089 IP (tos 0x0, ttl 1, id 51794, offset 0, flags [none], proto UDP (17), length 28)
    X.X.X.X.51792 > Y.Y.Y.Y.33437: [udp sum ok] UDP, length 0
16:03:30.442149 IP (tos 0xc0, ttl 64, id 56351, offset 0, flags [none], proto ICMP (1), length 56)
    A.A.A.A > X.X.X.X: ICMP time exceeded in-transit, length 36
        IP (tos 0x0, ttl 1, id 51794, offset 0, flags [none], proto UDP (17), length 28)
    X.X.X.X.51792 > Y.Y.Y.Y.33437: [udp sum ok] UDP, length 0

16:03:30.472133 IP (tos 0x0, ttl 2, id 51795, offset 0, flags [none], proto UDP (17), length 28)
    X.X.X.X.51792 > Y.Y.Y.Y.33438: [udp sum ok] UDP, length 0
16:03:30.473702 IP (tos 0x0, ttl 254, id 63400, offset 0, flags [none], proto ICMP (1), length 96)
    B.B.B.B > X.X.X.X: ICMP time exceeded in-transit, length 76
        IP (tos 0x0, ttl 1, id 51795, offset 0, flags [none], proto UDP (17), length 28)
    X.X.X.X.51792 > Y.Y.Y.Y.33438: [udp sum ok] UDP, length 0
16:03:30.501837 IP (tos 0x0, ttl 2, id 51796, offset 0, flags [none], proto UDP (17), length 28)
    X.X.X.X.51792 > Y.Y.Y.Y.33439: [udp sum ok] UDP, length 0
16:03:30.502958 IP (tos 0x0, ttl 254, id 63404, offset 0, flags [none], proto ICMP (1), length 96)
    B.B.B.B > X.X.X.X: ICMP time exceeded in-transit, length 76
        IP (tos 0x0, ttl 1, id 51796, offset 0, flags [none], proto UDP (17), length 28)
    X.X.X.X.51792 > Y.Y.Y.Y.33439: [udp sum ok] UDP, length 0
16:03:30.531875 IP (tos 0x0, ttl 2, id 51797, offset 0, flags [none], proto UDP (17), length 28)
    X.X.X.X.51792 > Y.Y.Y.Y.33440: [udp sum ok] UDP, length 0
16:03:30.532687 IP (tos 0x0, ttl 254, id 63406, offset 0, flags [none], proto ICMP (1), length 96)
    B.B.B.B > X.X.X.X: ICMP time exceeded in-transit, length 76
        IP (tos 0x0, ttl 1, id 51797, offset 0, flags [none], proto UDP (17), length 28)
    X.X.X.X.51792 > Y.Y.Y.Y.33440: [udp sum ok] UDP, length 0

16:03:30.561849 IP (tos 0x0, ttl 3, id 51798, offset 0, flags [none], proto UDP (17), length 28)
    X.X.X.X.51792 > Y.Y.Y.Y.33441: [udp sum ok] UDP, length 0
16:03:30.562477 IP (tos 0xc0, ttl 62, id 41919, offset 0, flags [none], proto ICMP (1), length 56)
    C.C.C.C > X.X.X.X: ICMP time exceeded in-transit, length 36
        IP (tos 0x0, ttl 1, id 51798, offset 0, flags [none], proto UDP (17), length 28)
    X.X.X.X.51792 > Y.Y.Y.Y.33441: [udp sum ok] UDP, length 0
16:03:30.591973 IP (tos 0x0, ttl 3, id 51799, offset 0, flags [none], proto UDP (17), length 28)
    X.X.X.X.51792 > Y.Y.Y.Y.33442: [udp sum ok] UDP, length 0
16:03:30.592394 IP (tos 0xc0, ttl 62, id 41922, offset 0, flags [none], proto ICMP (1), length 56)
    C.C.C.C > X.X.X.X: ICMP time exceeded in-transit, length 36
        IP (tos 0x0, ttl 1, id 51799, offset 0, flags [none], proto UDP (17), length 28)
    X.X.X.X.51792 > Y.Y.Y.Y.33442: [udp sum ok] UDP, length 0
16:03:30.622144 IP (tos 0x0, ttl 3, id 51800, offset 0, flags [none], proto UDP (17), length 28)
    X.X.X.X.51792 > Y.Y.Y.Y.33443: [udp sum ok] UDP, length 0
16:03:30.624016 IP (tos 0xc0, ttl 62, id 41924, offset 0, flags [none], proto ICMP (1), length 56)
    C.C.C.C > X.X.X.X: ICMP time exceeded in-transit, length 36
        IP (tos 0x0, ttl 1, id 51800, offset 0, flags [none], proto UDP (17), length 28)
    X.X.X.X.51792 > Y.Y.Y.Y.33443: [udp sum ok] UDP, length 0
{% endhighlight %}

As it can be seen from the above, Cisco IOS-XR default traceroute command uses UDP packets with TTL initially set to 1 and increased by 1 for every third probe. Cisco provides few options available that can be used to alter default traceroute values.

#### Cisco IOS-XE

Below is traceroute packet capture from a Cisco IOS-XE host:

{% highlight console %}
16:10:09.596549 IP (tos 0x0, ttl 1, id 11, offset 0, flags [none], proto UDP (17), length 28)
    X.X.X.X.49154 > Y.Y.Y.Y.33434: [udp sum ok] UDP, length 0
16:10:09.596595 IP (tos 0xc0, ttl 64, id 30741, offset 0, flags [none], proto ICMP (1), length 56)
    A.A.A.A > X.X.X.X: ICMP time exceeded in-transit, length 36
        IP (tos 0x0, ttl 1, id 11, offset 0, flags [none], proto UDP (17), length 28)
    X.X.X.X.49154 > Y.Y.Y.Y.33434: [udp sum ok] UDP, length 0
16:10:17.602565 IP (tos 0x0, ttl 1, id 14, offset 0, flags [none], proto UDP (17), length 28)
    X.X.X.X.49155 > Y.Y.Y.Y.33435: [udp sum ok] UDP, length 0
16:10:17.602609 IP (tos 0xc0, ttl 64, id 31056, offset 0, flags [none], proto ICMP (1), length 56)
    A.A.A.A > X.X.X.X: ICMP time exceeded in-transit, length 36
        IP (tos 0x0, ttl 1, id 14, offset 0, flags [none], proto UDP (17), length 28)
    X.X.X.X.49155 > Y.Y.Y.Y.33435: [udp sum ok] UDP, length 0
16:10:17.604560 IP (tos 0x0, ttl 1, id 15, offset 0, flags [none], proto UDP (17), length 28)
    X.X.X.X.49156 > Y.Y.Y.Y.33436: [udp sum ok] UDP, length 0
16:10:17.604590 IP (tos 0xc0, ttl 64, id 31057, offset 0, flags [none], proto ICMP (1), length 56)
    A.A.A.A > X.X.X.X: ICMP time exceeded in-transit, length 36
        IP (tos 0x0, ttl 1, id 15, offset 0, flags [none], proto UDP (17), length 28)
    X.X.X.X.49156 > Y.Y.Y.Y.33436: [udp sum ok] UDP, length 0

16:10:17.606545 IP (tos 0x0, ttl 2, id 16, offset 0, flags [none], proto UDP (17), length 28)
    X.X.X.X.49157 > Y.Y.Y.Y.33437: [udp sum ok] UDP, length 0
16:10:17.607363 IP (tos 0x0, ttl 254, id 17362, offset 0, flags [none], proto ICMP (1), length 96)
    B.B.B.B > X.X.X.X: ICMP time exceeded in-transit, length 76
        IP (tos 0x0, ttl 1, id 16, offset 0, flags [none], proto UDP (17), length 28)
    X.X.X.X.49157 > Y.Y.Y.Y.33437: [udp sum ok] UDP, length 0
16:10:25.611561 IP (tos 0x0, ttl 2, id 19, offset 0, flags [none], proto UDP (17), length 28)
    X.X.X.X.49158 > Y.Y.Y.Y.33438: [udp sum ok] UDP, length 0
16:10:25.612567 IP (tos 0x0, ttl 254, id 17126, offset 0, flags [none], proto ICMP (1), length 96)
    B.B.B.B > X.X.X.X: ICMP time exceeded in-transit, length 76
        IP (tos 0x0, ttl 1, id 19, offset 0, flags [none], proto UDP (17), length 28)
    X.X.X.X.49158 > Y.Y.Y.Y.33438: [udp sum ok] UDP, length 0
16:10:25.614552 IP (tos 0x0, ttl 2, id 20, offset 0, flags [none], proto UDP (17), length 28)
    X.X.X.X.49159 > Y.Y.Y.Y.33439: [udp sum ok] UDP, length 0
16:10:25.615399 IP (tos 0x0, ttl 254, id 17769, offset 0, flags [none], proto ICMP (1), length 96)
    B.B.B.B > X.X.X.X: ICMP time exceeded in-transit, length 76
        IP (tos 0x0, ttl 1, id 20, offset 0, flags [none], proto UDP (17), length 28)
    X.X.X.X.49159 > Y.Y.Y.Y.33439: [udp sum ok] UDP, length 0

16:10:25.617547 IP (tos 0x0, ttl 3, id 21, offset 0, flags [none], proto UDP (17), length 28)
    X.X.X.X.49160 > Y.Y.Y.Y.33440: [udp sum ok] UDP, length 0
16:10:25.618248 IP (tos 0xc0, ttl 62, id 7160, offset 0, flags [none], proto ICMP (1), length 56)
    198.27.73.95 > X.X.X.X: ICMP time exceeded in-transit, length 36
        IP (tos 0x0, ttl 1, id 21, offset 0, flags [none], proto UDP (17), length 28)
    X.X.X.X.49160 > Y.Y.Y.Y.33440: [udp sum ok] UDP, length 0
16:10:33.621573 IP (tos 0x0, ttl 3, id 24, offset 0, flags [none], proto UDP (17), length 28)
    X.X.X.X.49161 > Y.Y.Y.Y.33441: [udp sum ok] UDP, length 0
16:10:33.622029 IP (tos 0xc0, ttl 62, id 7697, offset 0, flags [none], proto ICMP (1), length 56)
    198.27.73.95 > X.X.X.X: ICMP time exceeded in-transit, length 36
        IP (tos 0x0, ttl 1, id 24, offset 0, flags [none], proto UDP (17), length 28)
    X.X.X.X.49161 > Y.Y.Y.Y.33441: [udp sum ok] UDP, length 0
16:10:33.623576 IP (tos 0x0, ttl 3, id 25, offset 0, flags [none], proto UDP (17), length 28)
    X.X.X.X.49162 > Y.Y.Y.Y.33442: [udp sum ok] UDP, length 0
16:10:33.623911 IP (tos 0xc0, ttl 62, id 7698, offset 0, flags [none], proto ICMP (1), length 56)
    198.27.73.95 > X.X.X.X: ICMP time exceeded in-transit, length 36
        IP (tos 0x0, ttl 1, id 25, offset 0, flags [none], proto UDP (17), length 28)
    X.X.X.X.49162 > Y.Y.Y.Y.33442: [udp sum ok] UDP, length 0
{% endhighlight %}

As can be seen from the above, Cisco IOS-XE default traceroute command uses UDP packets with TTL initially set to 1 and increased by 1 for every third probe. Cisco provides few options available that can be used to alter default traceroute values.

#### Juniper JunOS

Below is traceroute packet capture from a Juniper JunOS host:

{% highlight console %}
15:38:51.437825 IP (tos 0x0, ttl 1, id 36412, offset 0, flags [none], proto UDP (17), length 40)
    X.X.X.X.36411 > Y.Y.Y.Y.33434: [no cksum] UDP, length 12
15:38:51.437884 IP (tos 0xc0, ttl 64, id 21700, offset 0, flags [none], proto ICMP (1), length 68)
    A.A.A.A > X.X.X.X: ICMP time exceeded in-transit, length 48
        IP (tos 0x0, ttl 1, id 36412, offset 0, flags [none], proto UDP (17), length 40)
    X.X.X.X.36411 > Y.Y.Y.Y.33434: [no cksum] UDP, length 12
15:38:51.440245 IP (tos 0x0, ttl 1, id 36413, offset 0, flags [none], proto UDP (17), length 40)
    X.X.X.X.36411 > Y.Y.Y.Y.33435: [no cksum] UDP, length 12
15:38:51.440269 IP (tos 0xc0, ttl 64, id 21701, offset 0, flags [none], proto ICMP (1), length 68)
    A.A.A.A > X.X.X.X: ICMP time exceeded in-transit, length 48
        IP (tos 0x0, ttl 1, id 36413, offset 0, flags [none], proto UDP (17), length 40)
    X.X.X.X.36411 > Y.Y.Y.Y.33435: [no cksum] UDP, length 12
15:38:51.440817 IP (tos 0x0, ttl 1, id 36414, offset 0, flags [none], proto UDP (17), length 40)
    X.X.X.X.36411 > Y.Y.Y.Y.33436: [no cksum] UDP, length 12
15:38:51.440839 IP (tos 0xc0, ttl 64, id 21702, offset 0, flags [none], proto ICMP (1), length 68)
    A.A.A.A > X.X.X.X: ICMP time exceeded in-transit, length 48
        IP (tos 0x0, ttl 1, id 36414, offset 0, flags [none], proto UDP (17), length 40)
    X.X.X.X.36411 > Y.Y.Y.Y.33436: [no cksum] UDP, length 12

15:38:51.441569 IP (tos 0x0, ttl 2, id 36415, offset 0, flags [none], proto UDP (17), length 40)
    X.X.X.X.36411 > Y.Y.Y.Y.33437: [no cksum] UDP, length 12
15:38:51.442719 IP (tos 0x0, ttl 254, id 64781, offset 0, flags [none], proto ICMP (1), length 96)
    B.B.B.B > X.X.X.X: ICMP time exceeded in-transit, length 76
        IP (tos 0x0, ttl 1, id 36415, offset 0, flags [none], proto UDP (17), length 40)
    X.X.X.X.36411 > Y.Y.Y.Y.33437: [no cksum] UDP, length 12
15:38:51.444569 IP (tos 0x0, ttl 2, id 36416, offset 0, flags [none], proto UDP (17), length 40)
    X.X.X.X.36411 > Y.Y.Y.Y.33438: [no cksum] UDP, length 12
15:38:51.445804 IP (tos 0x0, ttl 254, id 62038, offset 0, flags [none], proto ICMP (1), length 96)
    B.B.B.B > X.X.X.X: ICMP time exceeded in-transit, length 76
        IP (tos 0x0, ttl 1, id 36416, offset 0, flags [none], proto UDP (17), length 40)
    X.X.X.X.36411 > Y.Y.Y.Y.33438: [no cksum] UDP, length 12
15:38:51.446543 IP (tos 0x0, ttl 2, id 36417, offset 0, flags [none], proto UDP (17), length 40)
    X.X.X.X.36411 > Y.Y.Y.Y.33439: [no cksum] UDP, length 12
15:38:51.447612 IP (tos 0x0, ttl 254, id 62039, offset 0, flags [none], proto ICMP (1), length 96)
    B.B.B.B > X.X.X.X: ICMP time exceeded in-transit, length 76
        IP (tos 0x0, ttl 1, id 36417, offset 0, flags [none], proto UDP (17), length 40)
    X.X.X.X.36411 > Y.Y.Y.Y.33439: [no cksum] UDP, length 12

15:38:51.448568 IP (tos 0x0, ttl 3, id 36418, offset 0, flags [none], proto UDP (17), length 40)
    X.X.X.X.36411 > Y.Y.Y.Y.33440: [no cksum] UDP, length 12
15:38:51.448988 IP (tos 0xc0, ttl 62, id 39390, offset 0, flags [none], proto ICMP (1), length 68)
    C.C.C.C > X.X.X.X: ICMP time exceeded in-transit, length 48
        IP (tos 0x0, ttl 1, id 36418, offset 0, flags [none], proto UDP (17), length 40)
    X.X.X.X.36411 > Y.Y.Y.Y.33440: [no cksum] UDP, length 12
15:38:51.451090 IP (tos 0x0, ttl 3, id 36419, offset 0, flags [none], proto UDP (17), length 40)
    X.X.X.X.36411 > Y.Y.Y.Y.33441: [no cksum] UDP, length 12
15:38:51.451507 IP (tos 0xc0, ttl 62, id 39391, offset 0, flags [none], proto ICMP (1), length 68)
    C.C.C.C > X.X.X.X: ICMP time exceeded in-transit, length 48
        IP (tos 0x0, ttl 1, id 36419, offset 0, flags [none], proto UDP (17), length 40)
    X.X.X.X.36411 > Y.Y.Y.Y.33441: [no cksum] UDP, length 12
15:38:51.452752 IP (tos 0x0, ttl 3, id 36420, offset 0, flags [none], proto UDP (17), length 40)
    X.X.X.X.36411 > Y.Y.Y.Y.33442: [no cksum] UDP, length 12
15:38:51.453348 IP (tos 0xc0, ttl 62, id 39392, offset 0, flags [none], proto ICMP (1), length 68)
    C.C.C.C > X.X.X.X: ICMP time exceeded in-transit, length 48
        IP (tos 0x0, ttl 1, id 36420, offset 0, flags [none], proto UDP (17), length 40)
    X.X.X.X.36411 > Y.Y.Y.Y.33442: [no cksum] UDP, length 12
{% endhighlight %}

As can be seen from the above, Juniper JunOS default traceroute command uses UDP packets with TTL initially set to 1 and increased by 1 for every third probe. Juniper provides few options available that can be used to alter default traceroute values.

#### Nokia TiMOS

Below is traceroute packet capture from a Nokia TiMOS host:

{% highlight console %}
16:30:38.034887 IP (tos 0x0, ttl 1, id 10, offset 0, flags [none], proto UDP (17), length 40)                                                      
    X.X.X.X.49157 > Y.Y.Y.Y.33435: [no cksum] UDP, length 12
16:30:38.034938 IP (tos 0xc0, ttl 64, id 20170, offset 0, flags [none], proto ICMP (1), length 68)
    A.A.A.A > X.X.X.X: ICMP time exceeded in-transit, length 48
        IP (tos 0x0, ttl 1, id 10, offset 0, flags [none], proto UDP (17), length 40)
    X.X.X.X.49157 > Y.Y.Y.Y.33435: [no cksum] UDP, length 12
16:30:38.238478 IP (tos 0x0, ttl 1, id 11, offset 0, flags [none], proto UDP (17), length 40)
    X.X.X.X.49157 > Y.Y.Y.Y.33436: [no cksum] UDP, length 12
16:30:38.238525 IP (tos 0xc0, ttl 64, id 20217, offset 0, flags [none], proto ICMP (1), length 68)
    A.A.A.A > X.X.X.X: ICMP time exceeded in-transit, length 48
        IP (tos 0x0, ttl 1, id 11, offset 0, flags [none], proto UDP (17), length 40)
    X.X.X.X.49157 > Y.Y.Y.Y.33436: [no cksum] UDP, length 12
16:30:38.448533 IP (tos 0x0, ttl 1, id 12, offset 0, flags [none], proto UDP (17), length 40)
    X.X.X.X.49157 > Y.Y.Y.Y.33437: [no cksum] UDP, length 12
16:30:38.448601 IP (tos 0xc0, ttl 64, id 20221, offset 0, flags [none], proto ICMP (1), length 68)
    A.A.A.A > X.X.X.X: ICMP time exceeded in-transit, length 48
        IP (tos 0x0, ttl 1, id 12, offset 0, flags [none], proto UDP (17), length 40)
    X.X.X.X.49157 > Y.Y.Y.Y.33437: [no cksum] UDP, length 12

16:30:38.658423 IP (tos 0x0, ttl 2, id 13, offset 0, flags [none], proto UDP (17), length 40)
    X.X.X.X.49157 > Y.Y.Y.Y.33438: [no cksum] UDP, length 12
16:30:38.659816 IP (tos 0x0, ttl 254, id 7103, offset 0, flags [none], proto ICMP (1), length 96)
    B.B.B.B > X.X.X.X: ICMP time exceeded in-transit, length 76
        IP (tos 0x0, ttl 1, id 13, offset 0, flags [none], proto UDP (17), length 40)
    X.X.X.X.49157 > Y.Y.Y.Y.33438: [no cksum] UDP, length 12
16:30:43.670013 IP (tos 0x0, ttl 2, id 14, offset 0, flags [none], proto UDP (17), length 40)
    X.X.X.X.49157 > Y.Y.Y.Y.33439: [no cksum] UDP, length 12
16:30:43.671118 IP (tos 0x0, ttl 254, id 7324, offset 0, flags [none], proto ICMP (1), length 96)
    B.B.B.B > X.X.X.X: ICMP time exceeded in-transit, length 76
        IP (tos 0x0, ttl 1, id 14, offset 0, flags [none], proto UDP (17), length 40)
    X.X.X.X.49157 > Y.Y.Y.Y.33439: [no cksum] UDP, length 12
16:30:48.677920 IP (tos 0x0, ttl 2, id 15, offset 0, flags [none], proto UDP (17), length 40)
    X.X.X.X.49157 > Y.Y.Y.Y.33440: [no cksum] UDP, length 12
16:30:48.679060 IP (tos 0x0, ttl 254, id 7559, offset 0, flags [none], proto ICMP (1), length 96)
    B.B.B.B > X.X.X.X: ICMP time exceeded in-transit, length 76
        IP (tos 0x0, ttl 1, id 15, offset 0, flags [none], proto UDP (17), length 40)
    X.X.X.X.49157 > Y.Y.Y.Y.33440: [no cksum] UDP, length 12

16:30:53.687490 IP (tos 0x0, ttl 3, id 16, offset 0, flags [none], proto UDP (17), length 40)
    X.X.X.X.49157 > Y.Y.Y.Y.33441: [no cksum] UDP, length 12
16:30:53.687944 IP (tos 0xc0, ttl 62, id 63903, offset 0, flags [none], proto ICMP (1), length 68)
    C.C.C.C > X.X.X.X: ICMP time exceeded in-transit, length 48
        IP (tos 0x0, ttl 1, id 16, offset 0, flags [none], proto UDP (17), length 40)
    X.X.X.X.49157 > Y.Y.Y.Y.33441: [no cksum] UDP, length 12
16:30:53.897390 IP (tos 0x0, ttl 3, id 17, offset 0, flags [none], proto UDP (17), length 40)
    X.X.X.X.49157 > Y.Y.Y.Y.33442: [no cksum] UDP, length 12
16:30:53.897928 IP (tos 0xc0, ttl 62, id 63906, offset 0, flags [none], proto ICMP (1), length 68)
    C.C.C.C > X.X.X.X: ICMP time exceeded in-transit, length 48
        IP (tos 0x0, ttl 1, id 17, offset 0, flags [none], proto UDP (17), length 40)
    X.X.X.X.49157 > Y.Y.Y.Y.33442: [no cksum] UDP, length 12
16:30:54.107343 IP (tos 0x0, ttl 3, id 18, offset 0, flags [none], proto UDP (17), length 40)
    X.X.X.X.49157 > Y.Y.Y.Y.33443: [no cksum] UDP, length 12
16:30:54.107865 IP (tos 0xc0, ttl 62, id 63910, offset 0, flags [none], proto ICMP (1), length 68)
    C.C.C.C > X.X.X.X: ICMP time exceeded in-transit, length 48
        IP (tos 0x0, ttl 1, id 18, offset 0, flags [none], proto UDP (17), length 40)
    X.X.X.X.49157 > Y.Y.Y.Y.33443: [no cksum] UDP, length 12
{% endhighlight %}

As can be seen from the above, Nokia TiMOS default traceroute command uses UDP packets with TTL initially set to 1 and increased by 1 for every third probe. Nokia provides few options available that can be used to alter default traceroute values.



> Note 1
> In the outputs above, note content of the ICMP time exceed in-transit packet.

> Note 2
> It is not shown in the outputs above, but how does a node detects that a traceroute response packet is from the final hop?
