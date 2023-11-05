---
title: "Configuring Flexible NetFlow on a Cisco Device"

date: 2022-12-20
url: /configuring-flexible-netflow-on-a-cisco-device/
image: images/2022-thumbs/configuring-flexible-netflow-on-a-cisco-device.png
categories:
  - Networking
tags:
  - Cisco
  - NetFlow
draft: false
-----

![configuring-flexible-netflow-on-a-cisco-device.png](/images/2022/configuring-flexible-netflow-on-a-cisco-device.png)

Flexible NetFlow is as the name states, a more flexible version of NetFlow. Flexible NetFlow gives us more granular control over the traffic flows that we want to see and what information we want to pull out of those flows.

There are three parts to configuring Flexible NetFlow:

* Flow Record
* Flow Exporter
* Flow Monitor

# Field Types
There are two field types in Flexible NetFlow:

1. Key fields
2. Non-key fields

A key field is a field that you want to match on which defines individual flows. If an incoming packet has a field that does not match a key field in the existing flow cache, a new flow entry is created.

A non-key field is extra information that can be added to existing flow record. Non-key fields are not used to identify flows (unlike key fields) but are instead used to add additional information to flows defined by the key fields.

# Configuring the Flow Record
The Flow Record defined the information that NetFlow will track, also known as the interesting traffic. Pre-defined flow record exist in IOS, however it is possible for user defined custom records to be configured. Flow Records are assigned to Flow Monitors to define where the flow data is stored.

Flow Records have two options:

1. Match
2. Collect

The match option defines the key fields that we want to group the flows with. The collect option defines the non-key fields to gather additional information to add to the flows.

Below is an example of a flow record. This record defines two key fields (IPv4 source address and IPv4 destination address), and one non-key field (Interface Output). Basically this flow record will create new entry in the flow cache whenever a new flow has a different value in one of the key fields (IPv4 source address and IPv4 destination address), if an entry already exist that matches the key fields, the flow information is added to that entry.

```
R1#sh run | s flow
flow record XOGS_RECORD
 match ipv4 source address
 match ipv4 destination address
 collect interface output
```

# Configuring the Flow Exporter
Flow Exporters define how the information collected by the flow record is sent to a NetFlow Collector such as Solarwinds Network Traffic Analyzer (NTA). By default Flexible NetFlow will use the Version 9 format for exported records, but it can be configured to support the Version 5 format.

In its simplest form, the Flow Exporter only needs the destination IP of the NetFlow Collector and the port that the Collector is listening on (2055 is the default for Solarwinds NTA).
```
R1#sh run | s flow exporter
flow exporter XOGS_EXPORTER
 destination 192.168.135.10
 transport udp 2055
```
# Configuring the Flow Monitor
The Flow Monitor links the Flow Record and Flow Exporter together, and is applied to the interfaces where we want to monitor flows. The monitor also defines how we want to cache flow information, in one of three ways:

1. Normal
2. Immediate
3. Permanent

In Normal mode entries in the cache are aged out according to the timeout active and timeout inactive settings. In Immediate mode entries in the cache are aged out as soon as the entry is created, this causes every flow to have only a one packet. In Permanent mode entries in the cache are never aged out. Note that if the cache type is not shown in the running config then the mode is set to Normal.

```
R1#sh run | s flow monitor
flow monitor XOGS_MONITOR
 exporter XOGS_EXPORTER
 record XOGS_RECORD
 cache type immediate

R1#sh run int gi0/1 | i monitor|Gi
interface GigabitEthernet0/1
 ip flow monitor XOGS_MONITOR input
 ip flow monitor XOGS_MONITOR output
```
# Viewing Flexible NetFlow Configuration
The below commands can be used to view information about the Flexible NetFlow configuration:

1. show flow monitor XOGS_MONITOR
2. show flow exporter XOGS_EXPORTER
3. show flow record name XOGS_RECORD
4. show flow monitor XOGS_MONITOR cache format table

```
R1#show flow monitor XOGS_MONITOR 
Flow Monitor XOGS_MONITOR:
  Description:       User defined
  Flow Record:       XOGS_RECORD
  Flow Exporter:     XOGS_EXPORTER
  Cache:
    Type:                 immediate
    Status:               allocated
    Size:                 4096 entries / 196620 bytes

R1#show flow exporter XOGS_EXPORTER 
Flow Exporter XOGS_EXPORTER:
  Description:              User defined
  Export protocol:          NetFlow Version 9
  Transport Configuration:
    Destination IP address: 192.168.135.10
    Source IP address:      192.168.135.1
    Transport Protocol:     UDP
    Destination Port:       2055
    Source Port:            56116
    DSCP:                   0x0
    TTL:                    255
    Output Features:        Not Used

R1#show flow record name XOGS_RECORD 
flow record XOGS_RECORD:
  Description:        User defined
  No. of users:       1
  Total field space:  12 bytes
  Fields:
    match ipv4 source address
    match ipv4 destination address
    collect interface output

R1#show flow monitor XOGS_MONITOR cache format table 
  Cache type:                            Immediate
  Cache size:                                 4096
  Current entries:                               0
  High Watermark:                                0
  Flows added:                                   0
    - Immediate timeout                          0

There are no cache entries to display.
```
