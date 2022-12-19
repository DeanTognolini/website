---
title: "Configuring Traditional NetFlow on a Cisco Device"

date: 2022-12-20
url: /configuring-traditional-netflow-on-a-cisco-device/
image: images/2022-thumbs/configuring-traditional-netflow-on-a-cisco-device.png
categories:
  - Networking
tags:
  - Cisco
  - Netflow
draft: false
-----

![configuring-traditional-netflow-on-a-cisco-device.png](/images/2022/configuring-traditional-netflow-on-a-cisco-device.png)

NetFlow is used to collect information about traffic flows in your environment, this information can be exported to a NetFlow analyzer which converts the raw data into human readable formats such as graphs, charts and tables. It is possible to configure NetFlow as a standalone feature on a network device and view the flow information from the local cache.

A network flow is a stream of packets between a source and destination that share the following matching fields in the IP header:

* Source IP
* Destination IP
* Source Port
* Destination Port
* Layer 3 Protocol Type
* Type of Service
* Input Interface

If any of these fields are different, it is a different flow.

When NetFlow is configured on a network device, the flow information is stored in a NetFlow cache entry in the local cache, an entry exists for each flow. When the flow expires the information is exported to a NetFlow collector if configured to do so, otherwise it is deleted.

# Configuring NetFlow
To configure NetFlow we must identify the interface where flows should be captured and enable NetFlow on that interface in either the ingress or egress direction.

To enable NetFlow on an interface the `ip flow [ ingress / egress ]` command is used.

```
R1(config)#int gi0/1.100
R1(config-if)#ip flow ingress 
R1(config-if)#end
```

To create some traffic for NetFlow to record I will send five pings from PC1 to PC2, the pings will be ingress on R1’s Gi0/1.100 sub-interface where NetFlow is enabled.
```
PC1> ping 192.168.200.10 
84 bytes from 192.168.200.10 icmp_seq=1 ttl=63 time=5.861 ms
84 bytes from 192.168.200.10 icmp_seq=2 ttl=63 time=2.876 ms
84 bytes from 192.168.200.10 icmp_seq=3 ttl=63 time=3.481 ms
84 bytes from 192.168.200.10 icmp_seq=4 ttl=63 time=2.829 ms
84 bytes from 192.168.200.10 icmp_seq=5 ttl=63 time=3.247 ms
```

# Viewing NetFlow Information
To view NetFlow information stored in the local cache the `show ip cache flow` command is used.

In the below output we can see there is 1 active flow, which is the pings from PC1 to PC2. NetFlow will hold onto information for inactive flows for 15 seconds, this timeout value can be see in the below output, if this command is ran within that time we can see information about the active flow at the bottom of the output. After 15 seconds this information will be deleted and added to the statistics table as shown in the second example of output, where we can see our 5 ICMP packets.

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
```
R1#show ip cache flow 
IP packet size distribution (5 total packets):
   1-32   64   96  128  160  192  224  256  288  320  352  384  416  448  480
   .000 .000 1.00 .000 .000 .000 .000 .000 .000 .000 .000 .000 .000 .000 .000
    512  544  576 1024 1536 2048 2560 3072 3584 4096 4608
   .000 .000 .000 .000 .000 .000 .000 .000 .000 .000 .000
IP Flow Switching Cache, 278544 bytes
  0 active, 4096 inactive, 1 added
  20 ager polls, 0 flow alloc failures
  Active flows timeout in 30 minutes
  Inactive flows timeout in 15 seconds
IP Sub Flow Cache, 34056 bytes
  0 active, 1024 inactive, 0 added, 0 added to flow
  0 alloc failures, 0 force free
  1 chunk, 0 chunks added
  last clearing of statistics 00:06:35
Protocol         Total    Flows   Packets Bytes  Packets Active(Sec) Idle(Sec)
--------         Flows     /Sec     /Flow  /Pkt     /Sec     /Flow     /Flow
ICMP                 1      0.0         5    84      0.0       4.0      15.2
Total:               1      0.0         5    84      0.0       4.0      15.2
SrcIf         SrcIPaddress    DstIf         DstIPaddress    Pr SrcP DstP  Pkts
```

# Clearing NetFlow Information
To clear NetFlow information stored in the local cache the clear ip flow stats command is used.

```
R1#clear ip flow stats 
R1#show ip cache flow  
IP packet size distribution (0 total packets):
   1-32   64   96  128  160  192  224  256  288  320  352  384  416  448  480
   .000 .000 .000 .000 .000 .000 .000 .000 .000 .000 .000 .000 .000 .000 .000
    512  544  576 1024 1536 2048 2560 3072 3584 4096 4608
   .000 .000 .000 .000 .000 .000 .000 .000 .000 .000 .000
IP Flow Switching Cache, 278544 bytes
  0 active, 4096 inactive, 0 added
  0 ager polls, 0 flow alloc failures
  Active flows timeout in 30 minutes
  Inactive flows timeout in 15 seconds
IP Sub Flow Cache, 34056 bytes
  0 active, 1024 inactive, 0 added, 0 added to flow
  0 alloc failures, 0 force free
  1 chunk, 0 chunks added
  last clearing of statistics 00:00:01
Protocol         Total    Flows   Packets Bytes  Packets Active(Sec) Idle(Sec)
--------         Flows     /Sec     /Flow  /Pkt     /Sec     /Flow     /Flow
SrcIf         SrcIPaddress    DstIf         DstIPaddress    Pr SrcP DstP  Pkts
```
