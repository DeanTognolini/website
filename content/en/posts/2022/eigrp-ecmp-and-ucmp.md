---
title: "EIGRP Multipathing"

date: 2022-03-01
url: /eigrp-ecmp-and-ucmp/
image: images/2022-thumbs/eigrp-ecmp-and-ucmp.webp
categories:
  - Networking
tags:
  - eigrp
draft: false
-----
By default EIGRP supports and is configured to use equal cost multi-pathing (ECMP). EIGRP also supports unequal cost multi-pathing (UCMP) but must be configured to use it.

# Equal Cost Multi-Pathing
First lets look at ECMP and how it behaves by default in EIGRP. We will be focusing on R4 and it’s routing towards the loopback interface on R1 with an IP address of 10.0.0.1/32. R4 has two EIGRP neighbours, R2 and R3, both of itss up-links have link delay values of 10.

![eigrp-ecmp-and-ucmp.webp](/images/2022/eigrp-ecmp-and-ucmp.webp)

If we check the EIGRP topology table on R4 for the 10.0.0.1/32 prefix we can see that we have two routes, one through R2 and the other through R3, notice that both of these routes have equal Feasible Distance (FD) values and EIGRP has assigned both of these routes as Successors (S).

```
R4#show eigrp address-family ipv4 topology 10.0.0.1/32
EIGRP-IPv4 Topology Entry for AS(1)/ID(10.0.0.4) for 10.0.0.1/32
  State is Passive, Query origin flag is 1, 2 Successor(s), FD is 131072
  Descriptor Blocks:
  10.1.24.2 (GigabitEthernet0/2), from 10.1.24.2, Send flag is 0x0
      Composite metric is (131072/130816), route is Internal
      Vector metric:
        Minimum bandwidth is 1000000 Kbit
        Total delay is 5020 microseconds
        Reliability is 255/255
        Load is 1/255
        Minimum MTU is 1500
        Hop count is 2
        Originating router is 10.0.0.1
  10.1.34.3 (GigabitEthernet0/0), from 10.1.34.3, Send flag is 0x0
      Composite metric is (131072/130816), route is Internal
      Vector metric:
        Minimum bandwidth is 1000000 Kbit
        Total delay is 5020 microseconds
        Reliability is 255/255
        Load is 1/255
        Minimum MTU is 1500
        Hop count is 2
        Originating router is 10.0.0.1
```

The RIB entry for the 10.0.0.1 prefix shows us additional information, specifically the traffic share count which tells us how flows are balanced between the two links. In ECMP’s case, it is equal load sharing meaning it is a one to one ratio.

```
R4#show ip route 10.0.0.1
Routing entry for 10.0.0.1/32
  Known via "eigrp 1", distance 90, metric 131072, type internal
  Redistributing via eigrp 1
  Last update from 10.1.34.3 on GigabitEthernet0/0, 00:17:52 ago
  Routing Descriptor Blocks:
    10.1.34.3, from 10.1.34.3, 00:17:52 ago, via GigabitEthernet0/0
      Route metric is 131072, traffic share count is 1
      Total delay is 5020 microseconds, minimum bandwidth is 1000000 Kbit
      Reliability 255/255, minimum MTU 1500 bytes
      Loading 1/255, Hops 2
  * 10.1.24.2, from 10.1.24.2, 00:17:52 ago, via GigabitEthernet0/2
      Route metric is 131072, traffic share count is 1
      Total delay is 5020 microseconds, minimum bandwidth is 1000000 Kbit
      Reliability 255/255, minimum MTU 1500 bytes
      Loading 1/255, Hops 2
```

We can find similar information in the Forwarding Information Base (FIB) table.

```
R4#show ip cef 10.0.0.1/32 internal 
10.0.0.1/32, epoch 0, RIB[I], refcnt 5, per-destination sharing
  sources: RIB 
  feature space:
    IPRM: 0x00028000
  ifnums:
    GigabitEthernet0/0(2): 10.1.34.3
    GigabitEthernet0/2(4): 10.1.24.2
  path list 1149D0F4, 5 locks, per-destination, flags 0x49 [shble, rif, hwcn]
    path 11532F0C, share 1/1, type attached nexthop, for IPv4
      nexthop 10.1.24.2 GigabitEthernet0/2, IP adj out of GigabitEthernet0/2, addr 10.1.24.2 10022DE0
    path 115330AC, share 1/1, type attached nexthop, for IPv4
      nexthop 10.1.34.3 GigabitEthernet0/0, IP adj out of GigabitEthernet0/0, addr 10.1.34.3 10022F10
  output chain:
    loadinfo 0FA419FC, per-session, 2 choices, flags 0083, 6 locks
      flags [Per-session, for-rx-IPv4, 2buckets]
      2 hash buckets
        < 0 > IP adj out of GigabitEthernet0/2, addr 10.1.24.2 10022DE0
        < 1 > IP adj out of GigabitEthernet0/0, addr 10.1.34.3 10022F10
      Subblocks:
        None
```

# Unequal Cost Multi-Pathing
In the below topology I have changed the delay value on Gi0/0 to 100, this will alter the FD value so that now Gi0/0 and Gi0/1 are no longer ECMP links.

![eigrp-ecmp-and-ucmp2.webp](/images/2022/eigrp-ecmp-and-ucmp2.webp)

Checking the RIB for the 10.0.0.1 prefix we can see that there is now only one path installed.

```
R4#conf t
*Jan  3 05:03:30.328: %SYS-5-CONFIG_I: Configured from console by cshow ip route 10.0.0.1
R4#show ip route 10.0.0.1
Routing entry for 10.0.0.1/32
  Known via "eigrp 1", distance 90, metric 131072, type internal
  Redistributing via eigrp 1
  Last update from 10.1.24.2 on GigabitEthernet0/2, 00:00:03 ago
  Routing Descriptor Blocks:
  * 10.1.24.2, from 10.1.24.2, 00:00:03 ago, via GigabitEthernet0/2
      Route metric is 131072, traffic share count is 1
      Total delay is 5020 microseconds, minimum bandwidth is 1000000 Kbit
      Reliability 255/255, minimum MTU 1500 bytes
      Loading 1/255, Hops 2
```

The EIGRP topology table on R4 now only has 1 Successor and the path via R3 is now a Feasible Successor (FS).

```
R4#show eigrp address-family ipv4 topology 10.0.0.1/32
EIGRP-IPv4 Topology Entry for AS(1)/ID(10.0.0.4) for 10.0.0.1/32
  State is Passive, Query origin flag is 1, 1 Successor(s), FD is 131072
  Descriptor Blocks:
  10.1.24.2 (GigabitEthernet0/2), from 10.1.24.2, Send flag is 0x0
      Composite metric is (131072/130816), route is Internal
      Vector metric:
        Minimum bandwidth is 1000000 Kbit
        Total delay is 5020 microseconds
        Reliability is 255/255
        Load is 1/255
        Minimum MTU is 1500
        Hop count is 2
        Originating router is 10.0.0.1
  10.1.34.3 (GigabitEthernet0/0), from 10.1.34.3, Send flag is 0x0
      Composite metric is (156416/130816), route is Internal
      Vector metric:
        Minimum bandwidth is 1000000 Kbit
        Total delay is 6010 microseconds
        Reliability is 255/255
        Load is 1/255
        Minimum MTU is 1500
        Hop count is 2
        Originating router is 10.0.0.1
```

Now lets enable UCMP in the EIGRP process on R4 and see how the RIB and EIGRP topology table changes.

```
R4(config)#router eigrp 1    
R4(config-router)#variance 2

R4#show ip route 10.0.0.1                             
Routing entry for 10.0.0.1/32
  Known via "eigrp 1", distance 90, metric 131072, type internal
  Redistributing via eigrp 1
  Last update from 10.1.34.3 on GigabitEthernet0/0, 00:00:14 ago
  Routing Descriptor Blocks:
    10.1.34.3, from 10.1.34.3, 00:00:14 ago, via GigabitEthernet0/0
      Route metric is 156416, traffic share count is 67
      Total delay is 6010 microseconds, minimum bandwidth is 1000000 Kbit
      Reliability 255/255, minimum MTU 1500 bytes
      Loading 1/255, Hops 2
  * 10.1.24.2, from 10.1.24.2, 00:00:14 ago, via GigabitEthernet0/2
      Route metric is 131072, traffic share count is 80
      Total delay is 5020 microseconds, minimum bandwidth is 1000000 Kbit
      Reliability 255/255, minimum MTU 1500 bytes
      Loading 1/255, Hops 2

R4#show eigrp address-family ipv4 topology 10.0.0.1/32
EIGRP-IPv4 Topology Entry for AS(1)/ID(10.0.0.4) for 10.0.0.1/32
  State is Passive, Query origin flag is 1, 2 Successor(s), FD is 131072
  Descriptor Blocks:
  10.1.24.2 (GigabitEthernet0/2), from 10.1.24.2, Send flag is 0x0
      Composite metric is (131072/130816), route is Internal
      Vector metric:
        Minimum bandwidth is 1000000 Kbit
        Total delay is 5020 microseconds
        Reliability is 255/255
        Load is 1/255
        Minimum MTU is 1500
        Hop count is 2
        Originating router is 10.0.0.1
  10.1.34.3 (GigabitEthernet0/0), from 10.1.34.3, Send flag is 0x0
      Composite metric is (156416/130816), route is Internal
      Vector metric:
        Minimum bandwidth is 1000000 Kbit
        Total delay is 6010 microseconds
        Reliability is 255/255
        Load is 1/255
        Minimum MTU is 1500
        Hop count is 2
        Originating router is 10.0.0.1
```

In the RIB we have two routes to 10.0.0.1, take note of the traffic share count values, this time they are not as simple as a 1:1 ratio. We are now doing unequal cost load balancing, EIGRP will automatically determine the appropriate balancing ratio depending on the metric of each link. The EIGRP topology tables tells us we have 2 Successors.

How did this work? UCMP is enabled by changing the variance value in the EIGRP process, which we did in the previous step. The variance value is a multiplier that tells EIGRP to multiple the Successor routes FD by the variance value and any Feasible Successor FD that is less than the result is a valid path to be used by UCMP.

To clarify further, in our example the Successor routes FD is 131072, the variance value is set to 2, 131072 times 2 is 262144, the Feasible Successor routes FD is 156416, 156416 is less than 262144 so it can be used by UCMP.

