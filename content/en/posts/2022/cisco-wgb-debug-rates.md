---
title: "Cisco WGB Debugging Data Rates"

date: 2022-12-15
url: /cisco-wgb-debug-rates/
image: images/2022-thumbs/cisco-logo.png
categories:
  - Networking
tags:
  - Cisco
  - Wireless
draft: false
-----

`debug dot11 <radio_interface> trace print rates` is a pretty cool command you can use to show debugging information related to the dynamic changing of WIFI MCS rates.

Below is an output from one of my lab APs while running this debug, the drop to M6 and M7 data rates is when I put a metal pot over the top of one of the APs!

```
WAP01#debug dot11 Dot11Radio 0 trace print rates 
*Mar  2 23:17:12.073: EC594601-0 4C093E - Set rate:    m15. 130 Mbps (10F), Rssi 25 dBm
*Mar  2 23:17:24.073: ED106AB8-0 4C093E - Set rate:    m14. 117 Mbps (10E), Rssi 25 dBm
*Mar  2 23:17:28.073: ED4D76F2-0 4C093E - Set rate:    m13. 104 Mbps (10D), Rssi 25 dBm
*Mar  2 23:17:30.073: ED6BFD92-0 4C093E - Set rate:    m14. 117 Mbps (10E), Rssi 25 dBm
*Mar  2 23:17:38.073: EDE615F5-0 4C093E - Set rate:    m13. 104 Mbps (10D), Rssi 25 dBm
*Mar  2 23:17:44.073: EE41A7DF-0 4C093E - Set rate:     m7.  65 Mbps (107), Rssi 25 dBm
*Mar  2 23:17:48.073: EE7EB41B-0 4C093E - Set rate:     m6.  59 Mbps (106), Rssi 25 dBm
*Mar  2 23:17:54.073: EEDA46E1-0 4C093E - Set rate:    m12.  78 Mbps (10C), Rssi 25 dBm
*Mar  2 23:18:00.073: EF35D8E0-0 4C093E - Set rate:    m13. 104 Mbps (10D), Rssi 25 dBm
WAP01#u all
All possible debugging has been turned off
```

This is pretty rudimentary but it is a useful way to quick check out what is going on with MCS rates.

You can also use the `show dot11 associations all-client` command to view the current MCS rate.

```
WAP01#show dot11 associations all-client 
Address           : bc16.654c.093e     Name             : WGB01
IP Address        : 10.1.100.12        Interface        : Dot11Radio 0
Device            : WGB                Software Version : NONE 
CCX Version       : 5                  Client MFP       : Off

State             : Assoc              Parent           : self               
SSID              : ISO_OT                          
VLAN              : 100
Hops to Infra     : 1                  Association Id   : 1
Clients Associated: 0                  Repeaters associated: 0
Tunnel Address    : 0.0.0.0
Key Mgmt type     : WPAv2 PSK          Encryption       : AES-CCMP
Current Rate      : m15.               Capability       : WMM ShortHdr ShortSlot
Supported Rates   : 12.0 24.0 m0. m1. m2. m3. m4. m5. m6. m7. m8. m9. m10. m11. m12. m13. m14. m15.
Voice Rates       : disabled           Bandwidth        : 20 MHz 
Signal Strength   : -25  dBm           Connected for    : 3881 seconds
Signal to Noise   : 62  dB            Activity Timeout : 28 seconds
Power-save        : Off                Last Activity    : 1 seconds ago
Apsd DE AC(s)     : NONE

Packets Input     : 763                Packets Output   : 21615     
Bytes Input       : 81095              Bytes Output     : 2028145   
Duplicates Rcvd   : 0                  Data Retries     : 1365      
Decrypt Failed    : 0                  RTS Retries      : 0         
MIC Failed        : 0                  MIC Missing      : 0         
Packets Redirected: 0                  Redirect Filtered: 0         
Session timeout   : 0 seconds
Reauthenticate in : never

```
