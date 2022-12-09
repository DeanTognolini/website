---
title: "A Practical Example of the Native VLAN"

date: 2021-08-10
url: /NativeVLAN/
image: images/2021-thumbs/nativeVLAN.webp
categories:
  - Windows
tags:
  - VLAN 
draft: false
-----

# Summary 
* On ingress frames (frames coming into the switchport): If the frame does not have a VLAN tag, it will be tagged with the configured Native VLAN on the switchport.
* On egress frames (frames going out of the switchport): If the frames VLAN tag MATCHES the configured Native VLAN on the switchport, the tag will be stripped and the frame will be forwarded with no VLAN tag.

# Example
The below diagram shows a poorly designed network. The Native VLAN between S2 and S3 is not the same, the Native VLAN on S2 Gi0/1 is configured to 100, and the Native VLAN on S3 Gi0/0 is configured to the default, 1.

![nativeVLAN](/images/2021/nativeVLAN.webp)

Now Cisco devices will scream at you if the Native VLAN does not match on a link, firstly STP will go into a Broken state on the switchport and block the link on that VLAN. You will also see CDP messages telling you there is a mismatch. To get this lab to work I had to disable CDP and STP on the switches. However, I have seen this exact design in multi-vendor environments with no errors showing.

```>:
*Aug 10 03:21:00.491: %SPANTREE-2-RECV_PVID_ERR: Received BPDU with inconsistent peer vlan id 1 on GigabitEthernet0/1 VLAN100.
*Aug 10 03:21:00.491: %SPANTREE-2-BLOCK_PVID_PEER: Blocking GigabitEthernet0/1 on VLAN0001. Inconsistent peer vlan.
*Aug 10 03:21:00.491: %SPANTREE-2-BLOCK_PVID_LOCAL: Blocking GigabitEthernet0/1 on VLAN0100. Inconsistent local vlan.

*Aug 10 03:22:04.233: %CDP-4-NATIVE_VLAN_MISMATCH: Native VLAN mismatch discovered on GigabitEthernet0/1 (100), with S3 GigabitEthernet0/0 (1).

S2#sh sp vl 1

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Gi0/0               Desg FWD 4         128.1    P2p 
Gi0/1               Desg BKN*4         128.2    P2p *PVID_Inc
```

The below pcaps shows the frame that was sent from VPC to R1, as described by the steps on the diagram.

In the first frame which is the frame leaving S3 Gi0/0 towards S2, we can see there is no 802.1Q header because the VLAN tag was stripped due to the VLAN ID matching the Native VLAN on the switchport.

![nativeVLAN2](/images/2021/nativeVLAN2.webp)

In the second frame which is the frame leaving the S2 Gi0/0 towards R1, we can see there is a 802.1Q header because the VLAN tag was NOT stripped due to the VLANID NOT matching the Native VLAN on the switchport.

![nativeVLAN3](/images/2021/nativeVLAN3.webp)
