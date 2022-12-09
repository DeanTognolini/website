---
title: "EIGRP Stub Routing"

date: 2022-03-01
url: /eigrp-stub-routing/
image: images/2022-thumbs/eigrp-stub-routing.webp
categories:
  - Networking
tags:
  - eigrp
draft: false
-----
In the interest of conserving local router resources and alleviating network load, EIGRP supports stub routing. A stub router is a router that should not be used as a transit router, meaning that the stub router does not have any information about other EIGRP routers behind it and does not need to advertise anything other than locally connected and summary routes to its neighbours.

# Lab
Before we configure R4 as a stub router, lets explore the default behaviour of EIGRP when a route goes down. In EIGRP when a Successor (S) route goes down a Feasible Successor (FS) route will instantly take over causing no loss of communications, however if no FS route exists or the FS route is also down then we have a problem. When this occurs EIGRP routers will flood Query messages to all neighbours, this behaviour has the potential to cause a lot of activity in a large EIGRP network and we should employ all techniques to contain this where possible, in this example, the technique is stub routing.

Lets see this occur in real time with the below topology, note that no stub routers have been configured yet. To simulate a lost route I am going to shutdown the Loopback0 interface with the IP address 2.2.2.2/32 on R2, as there are no FS routes for 2.2.2.2/32 this will force R2 to flood Queries to its neighbours. The EIGRP topology table for the 2.2.2.2/32 prefix on R2 is shown below.

```
R2#show eigrp address-family ipv4 topology 2.2.2.2/32
EIGRP-IPv4 Topology Entry for AS(1)/ID(2.2.2.2) for 2.2.2.2/32
  State is Passive, Query origin flag is 1, 1 Successor(s), FD is 128256
  Descriptor Blocks:
  0.0.0.0 (Loopback0), from Connected, Send flag is 0x0
      Composite metric is (128256/0), route is Internal
      Vector metric:
        Minimum bandwidth is 8000000 Kbit
        Total delay is 5000 microseconds
        Reliability is 255/255
        Load is 1/255
        Minimum MTU is 1514
        Hop count is 0
        Originating router is 2.2.2.2
```

![eigrp-stub-routing.webp](/images/2022/eigrp-stub-routing.webp)

I have ran the below commands on R2, R3 and R4 to provide us with information to following what occurs. The diagram above also depicts what should occur.

```
R2# debug eigrp packets query reply

R3# debug eigrp packets query reply

R4# debug eigrp packets query reply
```

Lets shut the Loopback0 interface on R2:

```
R2(config)#int lo0
R2(config-if)#shut
```

As expected, R2 en-queues and sends query messages on its Gi0/0, Gi0/1, and Gi0/2 interface towards all of its EIGRP neighbours.
```
R2#
*Jan  3 03:16:34.733: EIGRP: Enqueueing QUERY on Gi0/1 - paklen 0 tid 0 iidbQ un/rely 0/1 serno 94-94
*Jan  3 03:16:34.735: EIGRP: Sending QUERY on Gi0/1 - paklen 45 tid 0

*Jan  3 03:16:34.736: EIGRP: Enqueueing QUERY on Gi0/2 - paklen 0 tid 0 iidbQ un/rely 0/1 serno 94-94
*Jan  3 03:16:34.738: EIGRP: Sending QUERY on Gi0/2 - paklen 45 tid 0

*Jan  3 03:16:34.737: EIGRP: Enqueueing QUERY on Gi0/0 - paklen 0 tid 0 iidbQ un/rely 0/1 serno 94-94
*Jan  3 03:16:34.741: EIGRP: Sending QUERY on Gi0/0 - paklen 45 tid 0
```

This shows that by default EIGRP floods query messages to everyone when a route goes down. Lets contain this flooding behaviour by configuring R4 as a stub router. In the below topology we can see that R4 is connected to R2 and R3, there are no other EIGRP routers connected to R4 so there is no point in sending query messages to R4 as it has no neighbours downstream to query.

![eigrp-stub-routing2.webp](/images/2022/eigrp-stub-routing2.webp)  

Lets configure R4 as a stub router. It is worth nothing that configured an EIGRP process as a stub will force the process to reset and drop all built neighbours.

```
R4(config)#router eigrp 1
R4(config-router)#eigrp stub 

*Jan  3 03:22:30.318: %DUAL-5-NBRCHANGE: EIGRP-IPv4 1: Neighbor 10.1.34.3 (GigabitEthernet0/0) is down: peer info changed
*Jan  3 03:22:30.319: %DUAL-5-NBRCHANGE: EIGRP-IPv4 1: Neighbor 10.1.24.2 (GigabitEthernet0/2) is down: peer info changed
*Jan  3 03:22:32.917: %DUAL-5-NBRCHANGE: EIGRP-IPv4 1: Neighbor 10.1.34.3 (GigabitEthernet0/0) is up: new adjacency
*Jan  3 03:22:34.554: %DUAL-5-NBRCHANGE: EIGRP-IPv4 1: Neighbor 10.1.24.2 (GigabitEthernet0/2) is up: new adjacency
```

Now lets run the same test again and see the difference.

```
(config)#int lo0
R2(config-if)#shut
```

Again we see three en-queuing actions to Gi0/0, Gi0/1 and Gi0/2, however only two sending actions. A query message was not sent out of Gi0/2 because this is the interface towards R4 which is now configured as a stub router, EIGRP will not send query messages towards stub routers.

```
R2#
*Jan  3 03:24:02.197: EIGRP: Enqueueing QUERY on Gi0/2 - paklen 0 tid 0 iidbQ un/rely 0/1 serno 113-113

*Jan  3 03:24:02.198: EIGRP: Enqueueing QUERY on Gi0/0 - paklen 0 tid 0 iidbQ un/rely 0/1 serno 113-113
*Jan  3 03:24:02.200: EIGRP: Sending QUERY on Gi0/0 - paklen 45 tid 0

*Jan  3 03:24:02.201: EIGRP: Enqueueing QUERY on Gi0/1 - paklen 0 tid 0 iidbQ un/rely 0/1 serno 113-113
*Jan  3 03:24:02.203: EIGRP: Sending QUERY on Gi0/1 - paklen 45 tid 0
```
