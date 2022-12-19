---
title: "Configuring NetFlow With Solarwinds Network Traffic Analyzer"

date: 2022-12-20
url: /configuring-netflow-with-solarwinds-network-traffic-analyzer/
image: images/2022-thumbs/configuring-netflow-with-solarwinds-network-traffic-analyzer.png
categories:
  - Networking
tags:
  - Cisco
  - NetFlow
draft: false
-----
![configuring-netflow-with-solarwinds-network-traffic-analyzer.png](/images/2022/configuring-netflow-with-solarwinds-network-traffic-analyzer.png)

NetFlow is used to collect information about traffic flows in your environment, this information is exported to a NetFlow analyzer which converts the raw data into human readable formats such as graphs, charts and tables. It is possible to configure NetFlow as a standalone feature on a network device and view the flow information locally on the device, however it is better to configure a centralised NetFlow collector that can store and track historical data that produces longer term reports, such as SolarWinds Network Traffic Analyzer.

A network flow is a stream of packets between a source and destination that share the following matching fields in the IP header:

* Source IP
* Destination IP
* Source Port
* Destination Port
* Layer 3 Protocol Type
* Type of Service
* Input Logical Interface

If any of these fields are different, it is a different flow.

When NetFlow is configured on a network device, the flow information is stored in a NetFlow cache entry in the local cache, an entry exists for each flow. When the flow expires the information is exported to a NetFlow collector if configured to do so, otherwise it is deleted.

# Configuring NetFlow
To configure NetFlow we must identify the interface where flows should be captured and enable NetFlow on that interface in either the ingress or egress direction.

To enable NetFlow on a single interface the `ip flow [ ingress / egress ]` command is used. If you are wanting to monitor the flows on multiple interfaces, it is best practice to only enable ingress monitoring on those interfaces to prevent duplicate flows.

If your NetFlow Collector is not displaying flow information one thing to check is that you have both directions of the flow monitored, for example I have both directions on Gi0/1.100 monitored because I have used both the ingress and egress command. I could also achieve the same thing by monitoring the ingress flows on Gi0/1.100 and the ingress flows on Gi0/1.200, as this would still see both directions of the flow.

```
R1(config)#int gi0/1.100
R1(config-if)#ip flow ingress
R1(config-if)#ip flow egress  
R1(config-if)#end
```

# Verifying NetFlow Configuration
Before we setup the NetFlow Exporter to send the NetFlow information to a Collector, we should validate the configuration we applied.

To create some traffic for NetFlow to record I will send five pings from PC1 to PC2, the pings will be ingress on R1’s Gi0/1.100 sub-interface where NetFlow is enabled, and the return flow will be egress on the same interface.

```
PC1> ping 192.168.200.10 
84 bytes from 192.168.200.10 icmp_seq=1 ttl=63 time=5.861 ms
84 bytes from 192.168.200.10 icmp_seq=2 ttl=63 time=2.876 ms
84 bytes from 192.168.200.10 icmp_seq=3 ttl=63 time=3.481 ms
84 bytes from 192.168.200.10 icmp_seq=4 ttl=63 time=2.829 ms
84 bytes from 192.168.200.10 icmp_seq=5 ttl=63 time=3.247 ms
```

To view NetFlow information stored in the local cache the `show ip cache flow` command is used. If we can see flow information in the output our NetFlow configuration is working as intended.

```
R1#show ip cache flow 
IP packet size distribution (5 total packets):
   1-32   64   96  128  160  192  224  256  288  320  352  384  416  448  480
   .000 .000 1.00 .000 .000 .000 .000 .000 .000 .000 .000 .000 .000 .000 .000
    512  544  576 1024 1536 2048 2560 3072 3584 4096 4608
   .000 .000 .000 .000 .000 .000 .000 .000 .000 .000 .000
IP Flow Switching Cache, 278544 bytes
  1 active, 4095 inactive, 1 added
  11 ager polls, 0 flow alloc failures
  Active flows timeout in 30 minutes
  Inactive flows timeout in 15 seconds
IP Sub Flow Cache, 34056 bytes
  0 active, 1024 inactive, 0 added, 0 added to flow
  0 alloc failures, 0 force free
  1 chunk, 0 chunks added
  last clearing of statistics 00:06:13
Protocol         Total    Flows   Packets Bytes  Packets Active(Sec) Idle(Sec)
--------         Flows     /Sec     /Flow  /Pkt     /Sec     /Flow     /Flow
SrcIf         SrcIPaddress    DstIf         DstIPaddress    Pr SrcP DstP  Pkts
Gi0/1.100     192.168.100.10  Gi0/1.200     192.168.200.10  01 0000 0800     5
```

# Configuring the NetFlow Exporter
To configure R1 to export NetFlow information to our Solarwinds NTA Collector we need to tell it what source interface to use, the NetFlow version and the destination IP of the Collector.

The below config is for NetFlow v5:

```
ip flow-export source GigabitEthernet0/0
ip flow-export version 5
ip flow-export destination 192.168.135.10 2055
```

The below config is for NetFlow v9:

```
ip flow-export source GigabitEthernet0/0
ip flow-export version 9
ip flow-export destination 192.168.135.10 2055
```

Note that when R1 sends NetFlow information to Solarwinds, it is going to discard the information until we configure R1 as a managed node within Solarwinds NPM. Solarwinds will tell us this with the below event message.

“The NetFlow Receiver Service [Solarwinds-Host] is receiving a flow data stream from an unmanaged device (192.168.135.1). The flow data stream from 192.168.135.1 will be discarded. Please use Orion Node management to manage this IP address in order to process this flow data stream, or just use Manage this device.”

Once R1 is managed we can send some test traffic to see if it gets to the Collector.

```
PC1> ping 192.168.200.10 
84 bytes from 192.168.200.10 icmp_seq=1 ttl=63 time=5.861 ms
84 bytes from 192.168.200.10 icmp_seq=2 ttl=63 time=2.876 ms
84 bytes from 192.168.200.10 icmp_seq=3 ttl=63 time=3.481 ms
84 bytes from 192.168.200.10 icmp_seq=4 ttl=63 time=2.829 ms
84 bytes from 192.168.200.10 icmp_seq=5 ttl=63 time=3.247 ms
```

On R1 I have ran the debug ip flow export command, which tells us when NetFlow on R1 is sending information to the Collector.

```
*Aug  7 06:04:41.544: IPFLOW: Sending UDP export pak 147 to 192.168.135.10 port 2055
```

I am also running Wireshark on the Solarwinds host. We can see the packet coming inbound containing two PDUs, one for each direction of the flow.

![configuring-netflow-with-solarwinds-network-traffic-analyzer1.png](/images/2022/configuring-netflow-with-solarwinds-network-traffic-analyzer1.png)

On the Solarwinds NTA Summary page we can see that we have received NetFlow traffic from R1.

![configuring-netflow-with-solarwinds-network-traffic-analyzer2.png](/images/2022/configuring-netflow-with-solarwinds-network-traffic-analyzer2.png)

![configuring-netflow-with-solarwinds-network-traffic-analyzer3.png](/images/2022/configuring-netflow-with-solarwinds-network-traffic-analyzer3.png)

# R1 Configuration
```
R1#terminal length 0
R1#sh run
Building configuration...
Current configuration : 3742 bytes
!
! Last configuration change at 05:13:27 UTC Sat Aug 7 2021
!
version 15.6
service timestamps debug datetime msec
service timestamps log datetime msec
no service password-encryption
!
hostname R1
!
boot-start-marker
boot-end-marker
!
!
!
no aaa new-model
ethernet lmi ce
!
!
!
mmi polling-interval 60
no mmi auto-configure
no mmi pvc
mmi snmp-timeout 180
!
!
!
!
!
!
!
!
ip dhcp excluded-address 192.168.100.1
ip dhcp excluded-address 192.168.200.1
!
ip dhcp pool VLAN100
 network 192.168.100.0 255.255.255.0
 default-router 192.168.100.1 
!
ip dhcp pool VLAN200
 network 192.168.200.0 255.255.255.0
 default-router 192.168.200.1 
!
!
!
ip flow-cache timeout active 1
ip cef
no ipv6 cef
!
multilink bundle-name authenticated
!
!
!
!
!
redundancy
!
!
! 
!
!
!
!
!
!
!
!
!
!
!
!
interface GigabitEthernet0/0
 ip address 192.168.135.1 255.255.255.0
 duplex auto
 speed auto
 media-type rj45
!
interface GigabitEthernet0/1
 no ip address
 duplex auto
 speed auto
 media-type rj45
!
interface GigabitEthernet0/1.1
 encapsulation dot1Q 1 native
!
interface GigabitEthernet0/1.100
 encapsulation dot1Q 100
 ip address 192.168.100.1 255.255.255.0
 ip flow ingress
 ip flow egress
!
interface GigabitEthernet0/1.200
 encapsulation dot1Q 200
 ip address 192.168.200.1 255.255.255.0
!
interface GigabitEthernet0/2
 no ip address
 shutdown
 duplex auto
 speed auto
 media-type rj45
!
interface GigabitEthernet0/3
 no ip address
 shutdown
 duplex auto
 speed auto
 media-type rj45
!
ip forward-protocol nd
!
ip flow-export source GigabitEthernet0/0
ip flow-export version 5
ip flow-export destination 192.168.135.10 2055
!
no ip http server
no ip http secure-server
!
!
snmp-server community XOGS RO
snmp-server ifindex persist
snmp-server chassis-id 
!
!
control-plane
!
banner exec ^C
**************************************************************************
* IOSv is strictly limited to use for evaluation, demonstration and IOS  *
* education. IOSv is provided as-is and is not supported by Cisco's      *
* Technical Advisory Center. Any use or disclosure, in whole or in part, *
* of the IOSv Software or Documentation to any third party for any       *
* purposes is expressly prohibited except as otherwise authorized by     *
* Cisco in writing.                                                      *
**************************************************************************^C
banner incoming ^C
**************************************************************************
* IOSv is strictly limited to use for evaluation, demonstration and IOS  *
* education. IOSv is provided as-is and is not supported by Cisco's      *
* Technical Advisory Center. Any use or disclosure, in whole or in part, *
* of the IOSv Software or Documentation to any third party for any       *
* purposes is expressly prohibited except as otherwise authorized by     *
* Cisco in writing.                                                      *
**************************************************************************^C
banner login ^C
**************************************************************************
* IOSv is strictly limited to use for evaluation, demonstration and IOS  *
* education. IOSv is provided as-is and is not supported by Cisco's      *
* Technical Advisory Center. Any use or disclosure, in whole or in part, *
* of the IOSv Software or Documentation to any third party for any       *
* purposes is expressly prohibited except as otherwise authorized by     *
* Cisco in writing.                                                      *
**************************************************************************^C
!
line con 0
line aux 0
line vty 0 4
 login
 transport input none
!
no scheduler allocate
!
end
```

# S1 Configuration
```
S1#sh run
Building configuration...

Current configuration : 3435 bytes
!
! Last configuration change at 23:09:57 UTC Fri Aug 6 2021
!
version 15.2
service timestamps debug datetime msec
service timestamps log datetime msec
no service password-encryption
service compress-config
!
hostname S1
!
boot-start-marker
boot-end-marker
!
!
!
no aaa new-model
!
!
!
!
!
no ip routing
!
!
!
no ip cef
no ipv6 cef
!
!
!
spanning-tree mode pvst
spanning-tree extend system-id
!
vlan internal allocation policy ascending
!
! 
!
!
!
!
!
!
!
!
!
!
!
!
interface GigabitEthernet0/0
 switchport trunk encapsulation dot1q
 switchport mode trunk
 media-type rj45
 negotiation auto
 spanning-tree portfast network
!
interface GigabitEthernet0/1
 switchport access vlan 100
 switchport mode access
 media-type rj45
 negotiation auto
 spanning-tree portfast edge
!
interface GigabitEthernet0/2
 switchport access vlan 200
 switchport mode access
 media-type rj45
 negotiation auto
 spanning-tree portfast edge
!
interface GigabitEthernet0/3
 switchport access vlan 200
 switchport mode access
 media-type rj45
 negotiation auto
 spanning-tree portfast edge
!
interface GigabitEthernet1/0
 media-type rj45
 negotiation auto
!
interface GigabitEthernet1/1
 media-type rj45
 negotiation auto
!
interface GigabitEthernet1/2
 media-type rj45
 negotiation auto
!
interface GigabitEthernet1/3
 media-type rj45
 negotiation auto
!
interface Vlan1
 ip address 192.168.1.2 255.255.255.0
 no ip route-cache
!
ip default-gateway 192.168.1.1
ip forward-protocol nd
!
no ip http server
no ip http secure-server
!
!
!
!
!
!
control-plane
!
banner exec ^C
**************************************************************************
* IOSv is strictly limited to use for evaluation, demonstration and IOS  *
* education. IOSv is provided as-is and is not supported by Cisco's      *
* Technical Advisory Center. Any use or disclosure, in whole or in part, *
* of the IOSv Software or Documentation to any third party for any       *
* purposes is expressly prohibited except as otherwise authorized by     *
* Cisco in writing.                                                      *
**************************************************************************^C
banner incoming ^C
**************************************************************************
* IOSv is strictly limited to use for evaluation, demonstration and IOS  *
* education. IOSv is provided as-is and is not supported by Cisco's      *
* Technical Advisory Center. Any use or disclosure, in whole or in part, *
* of the IOSv Software or Documentation to any third party for any       *
* purposes is expressly prohibited except as otherwise authorized by     *
* Cisco in writing.                                                      *
**************************************************************************^C
banner login ^C
**************************************************************************
* IOSv is strictly limited to use for evaluation, demonstration and IOS  *
* education. IOSv is provided as-is and is not supported by Cisco's      *
* Technical Advisory Center. Any use or disclosure, in whole or in part, *
* of the IOSv Software or Documentation to any third party for any       *
* purposes is expressly prohibited except as otherwise authorized by     *
* Cisco in writing.                                                      *
**************************************************************************^C
!
line con 0
line aux 0
line vty 0 4
 login
!
!
end
```
