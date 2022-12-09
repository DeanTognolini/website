---
title: "eBGP Multihop"

date: 2022-12-09
url: /ebgpmultihop/
image: images/2022-thumbs/ebgpmultihop.jpg
categories:
  - Networking
tags:
  - BGP
  - Routing
draft: false
-----

# Summary
* The `bgp neighbour X.X.X.X ebgp-multihop <value>` command is used to set the multihop value
* eBGP messages are sent with a TTL of 1 by default
* The `show ip bgp neighbors | i External` command is used to check the ebgp-multihop configuration
* The multihop requirement does not affect iBGP peerings

# eBGP Multihop
![ebgpmultihop1](/images/2022/ebgpmultihop1.png)

By default when configuring an eBGP neighbour, BGP expects the neighbours IP address to be less than 1 network hop away, or in other words, have a TTL of 1. BGP Multihop is used when the configured BGP neighbour IP address is greater than one hop away.

In this example I have configured each BGP router with a Loopback 1 interface, the IP addresses configured on these interfaces will be used to form a BGP session between each neighbour. The Loopback interfaces are greater than 1 hop away because they are not directly connected, there is one hop to the neighbouring routers directly connected interface, and then another hop "inside" of the router to reach the Loopback.

Because the neighbours Loopback interface are not directly connected we need a way for the routers to learn how to get to them. I have achieved this by creating OSPF adjacencies between the three routers and advertising the Loopback interfaces into OSPF. 

The BGP command `neighbor X.X.X.X update-source Loopback1` sets the Loopback1 interface as the source interface for any BGP messages sent to this neighbour. Because we are using Loopback1 as our BGP source interface for our peering with R2, our BGP messages will traverse 2 hops as depicted in the diagram below.

![ebgpmultihop2](/images/2022/ebgpmultihop2.png)

The BGP command `bgp neighbour X.X.X.X ebgp-multihop <value>` is used to increase the TTL of BGP messages sent to this neighbour. Below is a packet capture of a BGP TCP SYN message displaying a TTL of 2, which is the minimum numbers of hops required to establish a BGP peering in our topology. To set this value I used the `neighbour 2.2.2.2 ebgp-multihop 2` command on R1 and the `neighbour 1.1.1.1 ebgp-multihop 2` on R2. If you were to look at a packet capture of a default BGP ebgp-multihop configuration the TTL value would show 1.

![ebgpmultihop2](/images/2022/ebgpmultihop3.png)

The hop value can be confirmed with the `show ip bgp neighbors | i External` command.
```
R1#show ip bgp neighbors | i External
  External BGP neighbor may be up to 2 hops away.
```
# Configurations
## R1
```
!
hostname R1
!
interface Loopback1
 ip address 1.1.1.1 255.255.255.0
 ip ospf 1 area 0
!
interface GigabitEthernet0/0
 ip address 10.0.12.1 255.255.255.0
 ip ospf 1 area 0
!
interface GigabitEthernet0/1
 ip address 10.0.13.1 255.255.255.0
 ip ospf 1 area 0
!
router ospf 1
 router-id 1.1.1.1
 passive-interface default
 no passive-interface GigabitEthernet0/0
 no passive-interface GigabitEthernet0/1
!
router bgp 65001
 bgp router-id 1.1.1.1
 neighbor 2.2.2.2 remote-as 65002
 neighbor 2.2.2.2 ebgp-multihop 2
 neighbor 2.2.2.2 update-source Loopback1
 neighbor 3.3.3.3 remote-as 65003
 neighbor 3.3.3.3 ebgp-multihop 2
 neighbor 3.3.3.3 update-source Loopback1
 !
 address-family ipv4
  network 1.1.1.0 mask 255.255.255.0
  neighbor 2.2.2.2 activate
  neighbor 3.3.3.3 activate
 exit-address-family
!
```

## R2
```
!
hostname R2
!
interface Loopback1
 no shutdown
 ip address 2.2.2.2 255.255.255.0
 ip ospf 1 area 0
!
interface GigabitEthernet0/0
 ip address 10.0.12.2 255.255.255.0
 ip ospf 1 area 0
 media-type rj45
!
interface GigabitEthernet0/2
 ip address 10.0.23.2 255.255.255.0
 ip ospf 1 area 0
!
router ospf 1
 router-id 2.2.2.2
 passive-interface default
 no passive-interface GigabitEthernet0/0
 no passive-interface GigabitEthernet0/2
!
router bgp 65002
 bgp router-id 2.2.2.2
 neighbor 1.1.1.1 remote-as 65001
 neighbor 1.1.1.1 ebgp-multihop 2
 neighbor 1.1.1.1 update-source Loopback1
 neighbor 3.3.3.3 remote-as 65003
 neighbor 3.3.3.3 ebgp-multihop 2
 neighbor 3.3.3.3 update-source Loopback1
 !
 address-family ipv4
  network 2.2.2.0 mask 255.255.255.0
  neighbor 1.1.1.1 activate
  neighbor 3.3.3.3 activate
 exit-address-family
!
```

## R3
```
!
hostname R3
!
interface Loopback1
 ip address 3.3.3.3 255.255.255.0
 ip ospf 1 area 0
!
interface GigabitEthernet0/1
 ip address 10.0.13.3 255.255.255.0
 ip ospf 1 area 0
!
interface GigabitEthernet0/2
 ip address 10.0.23.3 255.255.255.0
 ip ospf 1 area 0
!
router ospf 1
 router-id 3.3.3.3
 passive-interface default
 no passive-interface GigabitEthernet0/1
 no passive-interface GigabitEthernet0/2
!
router bgp 65003
 bgp router-id 3.3.3.3
 neighbor 1.1.1.1 remote-as 65001
 neighbor 1.1.1.1 ebgp-multihop 2
 neighbor 1.1.1.1 update-source Loopback1
 neighbor 2.2.2.2 remote-as 65002
 neighbor 2.2.2.2 ebgp-multihop 2
 neighbor 2.2.2.2 update-source Loopback1
 !
 address-family ipv4
  network 3.3.3.0 mask 255.255.255.0
  neighbor 1.1.1.1 activate
  neighbor 2.2.2.2 activate
 exit-address-family
!
```
