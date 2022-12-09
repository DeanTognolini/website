---
title: "EIGRP Path Selection"

date: 2022-03-01
url: /eigrp-path-selection/
image: images/2022-thumbs/eigrp-path-selection.webp
categories:
  - Networking
tags:
  - eigrp
draft: false
-----

# Terminology
* Successor (S) – path with the lowest cost to the prefix.
* Feasible Successor (FS) – backup paths for S.
* Reported Distance (RD) – a neighbours distance to the prefix.
* Feasible Distance (FD) – the local routers distance from the prefix, e.g. it is the RD plus the local path cost of the link to the neighbouring router that shared the prefix.

# Lab
![eigrp-path-selection.webp](/images/2022/eigrp-path-selection.webp)

The above topology consists of three routers all speaking EIGRP on AS 1, each router has an EIGRP adjacency with both adjacent routers. R1 has learnt about the 3.3.3.3/32 prefix directly from R3 and also from R2. The below output shows the routes learnt from EIGRP neighbours for the 3.3.3.3/32 prefix.

```
R1#show eigrp address-family ipv4 topology 3.3.3.3/32
EIGRP-IPv4 Topology Entry for AS(1)/ID(1.1.1.1) for 3.3.3.3/32
  State is Passive, Query origin flag is 1, 1 Successor(s), FD is 130816
  Descriptor Blocks:
  10.1.13.3 (GigabitEthernet0/2), from 10.1.13.3, Send flag is 0x0
      Composite metric is (130816/128256), route is Internal
      Vector metric:
        Minimum bandwidth is 1000000 Kbit
        Total delay is 5010 microseconds
        Reliability is 255/255
        Load is 1/255
        Minimum MTU is 1500
        Hop count is 1
        Originating router is 3.3.3.3
  10.1.12.2 (GigabitEthernet0/0), from 10.1.12.2, Send flag is 0x0
      Composite metric is (131072/130816), route is Internal
      Vector metric:
        Minimum bandwidth is 1000000 Kbit
        Total delay is 5020 microseconds
        Reliability is 255/255
        Load is 1/255
        Minimum MTU is 1500
        Hop count is 2
        Originating router is 3.3.3.3
```

If we look at the RIB of R1 we can see that only the route through R3 has been installed, this is the S route to 3.3.3.3/32. This route has been selected as the S route because it has the lowest FD in the EIGRP topology table (130816 is lower than 131072).

```
R1#sh ip route eigrp | section 3.3.3.3
D        3.3.3.3 [90/130816] via 10.1.13.3, 00:04:34, GigabitEthernet0/2
      10.0.0.0/8 is variably subnetted, 5 subnets, 2 masks
```

Notice that the route via R2 has not been installed. The reason for this is the route through R2 has not passed the test to become a FS. To become a FS, the route must first pass the Feasibility Condition test where the RD of the route must be less than the FD of the S.

**FS Eligibility = RD < FD**

If we apply the Feasibility Condition logic to the advertisement for 3.3.3.3/32 coming from R2 we get the following:

```
R1#show eigrp address-family ipv4 topology 3.3.3.3/32 | section 10.1.12.2
  10.1.12.2 (GigabitEthernet0/0), from 10.1.12.2, Send flag is 0x0
      Composite metric is (131072/130816), route is Internal
      Vector metric:
        Minimum bandwidth is 1000000 Kbit
        Total delay is 5020 microseconds
        Reliability is 255/255
        Load is 1/255
        Minimum MTU is 1500
        Hop count is 2
        Originating router is 3.3.3.3
R1#sh ip route eigrp | section 3.3.3.3
D        3.3.3.3 [90/130816] via 10.1.13.3, 00:04:34, GigabitEthernet0/2
      10.0.0.0/8 is variably subnetted, 5 subnets, 2 masks
RD of the route to 3.3.3.3/32 from R2 = 130816
FD of the S path to 3.3.3.3/32 on R1 = 130816
RD < FD = False
```

The route does not pass the Feasibility Condition test and is not installed into the RIB as a FS.

# Influencing the EIGRP path cost
EIGRP uses the bandwidth and delay values of a link to determine the distance (by default). We can manipulate EIGRP path selection by altering these values. Lets manipulate the path cost of the R1 to R3 link to see if we can make R2 the S for the 3.3.3.3/32 prefix.

If we increase the delay of the R1 to R3 link this will increase the distance values, EIGRP will demote it as the S route and promote the R1 to R2 link instead.

The default delay value of the GigabitEthernet0/2 interface is 10 usec. If we change this to 100 usec that should be enough to make the distance worse than the R2 link.

```
R1#show interfaces gigabitEthernet 0/2 | i DLY  
  MTU 1500 bytes, BW 1000000 Kbit/sec, DLY 10 usec, 

R1(config)#int gi0/2   
R1(config-if)#delay 100

R1#show interfaces gigabitEthernet 0/2 | i DLY        
  MTU 1500 bytes, BW 1000000 Kbit/sec, DLY 1000 usec, 
```

If we check the RIB again we can see that the path via R2 is now the S route and the path via R3 has been dropped from the RIB.

```
R1#sh ip ro eigrp | s 3.3.3.3
D        3.3.3.3 [90/131072] via 10.1.12.2, 00:01:31, GigabitEthernet0/0
      10.0.0.0/8 is variably subnetted, 5 subnets, 2 masks
```

Looking at the EIGRP topology table we can also see that the distances for the R3 link have been changed.

```
EIGRP-IPv4 Topology Entry for AS(1)/ID(1.1.1.1) for 3.3.3.3/32
  State is Passive, Query origin flag is 1, 1 Successor(s), FD is 131072
  Descriptor Blocks:
  10.1.12.2 (GigabitEthernet0/0), from 10.1.12.2, Send flag is 0x0
      Composite metric is (131072/130816), route is Internal
      Vector metric:
        Minimum bandwidth is 1000000 Kbit
        Total delay is 5020 microseconds
        Reliability is 255/255
        Load is 1/255
        Minimum MTU is 1500
        Hop count is 2
        Originating router is 3.3.3.3
  10.1.13.3 (GigabitEthernet0/2), from 10.1.13.3, Send flag is 0x0
      Composite metric is (156160/128256), route is Internal
      Vector metric:
        Minimum bandwidth is 1000000 Kbit
        Total delay is 6000 microseconds
        Reliability is 255/255
        Load is 1/255
        Minimum MTU is 1500
        Hop count is 1
        Originating router is 3.3.3.3
```
