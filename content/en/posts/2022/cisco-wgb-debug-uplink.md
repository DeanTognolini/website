---
title: "Cisco WGB Debugging Uplink"

date: 2022-12-15
url: /cisco-wgb-debug-uplink/
image: images/2022-thumbs/cisco-logo.png
categories:
  - Networking
tags:
  - Cisco
  - Wireless
draft: false
-----

The `debug dot11 <radio_interface> trace print uplink` command can be used to see detailed information on what the radio interface is doing while looking for a parent.

The below shows the output from a WGB client AP when it lost its parent AP. We can see each step the client takes to associate to the AP.

```
*Apr 13 23:02:03.807: E60F534-0 Uplink: Increase scan wait time 23 msec
*Apr 13 23:02:20.663: F6235DD-0 Uplink: Increase scan wait time 28 msec
*Apr 13 23:02:24.798: FA15475-0 Uplink: Rcvd response from d0c2.823b.86c0 channel 6 589
*Apr 13 23:02:24.926: FA34753-0 Uplink: dot11_uplink_scan_done: rsnie_accept returns 0x0 key_mgmt 0xFAC02 encrypt_type 0x200
*Apr 13 23:02:24.926: FA34771-0 Uplink: ssid LAB_SSID auth open
*Apr 13 23:02:24.927: FA3477C-0 Uplink: try d0c2.823b.86c0, enc 200 key 4, priv 1, eap 0
*Apr 13 23:02:24.927: FA3478A-0 Uplink: Authenticating
*Apr 13 23:02:24.927: FA349DF-0 Uplink: Associating
*Apr 13 23:02:24.933: FA35AC5-0 Uplink: EAP authenticating
*Apr 13 23:02:24.940: %DOT11-4-UPLINK_ESTABLISHED: Interface Dot11Radio0, Associated To AP WAP01 d0c2.823b.86c0 [None WPAv2 PSK]
*Apr 13 23:02:24.941: %LINK-3-UPDOWN: Interface Dot11Radio0, changed state to up
*Apr 13 23:02:24.943: FA37E7D-0 Uplink: Done
*Apr 13 23:02:24.943: FA37EA6-0 Interface up
*Apr 13 23:02:25.941: %LINEPROTO-5-UPDOWN: Line protocol on Interface Dot11Radio0, changed state to up
```
