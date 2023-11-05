---
title: "Cisco WGB Multiple VLAN Configuration"

date: 2022-12-15
url: /cisco-wgb-config/
image: images/2022-thumbs/cisco-wgb-config1.png
categories:
  - Networking
tags:
  - Cisco
  - Wireless
draft: false
-----

These Cisco WGB configurations enable support for WGB-clients in different VLANs.

In this deployment I have two VLANs:

VLAN100 is for device management such as the AP BVI interfaces.
VLAN200 is for the WGB-client connected behind WGB01.

There is no switch behind the WGB, only a single directly connected ethernet client. The *workgroup-bridge client-vlan 200* command forces all WGB-clients onto VLAN200. If you had a dot1Q capable switch connected to the WGB you would not need this command, and instead use dot1Q tagged frames.

![cisco-wgb-config1.png](/images/2022/cisco-wgb-config1.png)

# Root AP
```
hostname WAP01
!
dot11 ssid LAB_SSID
   vlan 100
   authentication open
   authentication key-management wpa version 2
   wpa-psk ascii 12345678
   no ids mfp client
   infrastructure-ssid <-- makes the SSID an Infrastructure SSID
!
interface Dot11Radio0
 description 2.4GHz_CLIENT_ACCESS
 no ip address
 !
 encryption mode ciphers aes-ccm
 !
 encryption vlan 100 mode ciphers aes-ccm
 !
 ssid LAB_SSID
 !
 channel 2412
 station-role root <-- sets the radio to Root mode
 !
 infrastructure-client <-- makes the WGB an Infrastructure Client so it can associate to the Infrastructure SSID. Also used for reliable multicast.
!
interface Dot11Radio0.100
 description MGMT_INFRA_BG-1_2.4GHz_RADIO
 encapsulation dot1Q 100 native <-- native VLAN is always mapped to bridge-group 1
 bridge-group 1
!
interface Dot11Radio0.200
 description PROD_ONBOARD_BG-200_2.4GHz_RADIO
 encapsulation dot1Q 200
 bridge-group 200
!
interface GigabitEthernet0
 description LINK - CSW01 Gi1/0/8
!
interface GigabitEthernet0.100
 description MGMT_INFRA_BG-1_ETHERNET
 encapsulation dot1Q 100 native <-- native VLAN is always mapped to bridge-group 1
 bridge-group 1
!
interface GigabitEthernet0.200
 description PROD_ONBOARD_CAT_BG-200_ETHERNET
 encapsulation dot1Q 200
 bridge-group 200
!
interface BVI1
 description MGMT_INFRA_BG-1_AP
 ip address 10.1.100.10 255.255.255.0
!
ip default-gateway 10.1.100.1
!
end
```

# Work Group Bridge
```
hostname WGB01
!
dot11 ssid LAB_SSID
   vlan 100
   authentication open
   authentication key-management wpa version 2
   wpa-psk ascii 12345678
   no ids mfp client
   infrastructure-ssid <-- makes the SSID an Infrastructure SSID
!
interface Dot11Radio0
 description 2.4GHz_CLIENT_ACCESS
 no ip address
 !
 encryption mode ciphers aes-ccm
 !
 encryption vlan 100 mode ciphers aes-ccm
 !
 ssid LAB_SSID
 !
 station-role workgroup-bridge <-- sets the radio to WGB mode
 mobile station scan 2412 <-- sets the channel the WGB will scan for a parent
 mobile station minimum-rate 12.0 <-- sets the minimum basic rate
 mobile station period 20 threshold 70 <-- sets the RSSI monitoring parameters
 !
!
interface Dot11Radio0.100
 description MGMT_INFRA_BG-1_2.4GHz_RADIO
 encapsulation dot1Q 100 native <-- native VLAN is always mapped to bridge-group 1
 bridge-group 1
!
interface Dot11Radio0.200
 description PROD_ONBOARD_BG-200_2.4GHz_RADIO
 encapsulation dot1Q 200
 bridge-group 200
!
interface GigabitEthernet0
 description LINK - ONBOARD_EQUIPMENT
!
interface GigabitEthernet0.200
 description PROD_ONBOARD_BG-200_ETHERNET
 encapsulation dot1Q 200
 bridge-group 200
!
interface BVI1
 description MGMT_INFRA_BG-1_AP
 ip address 10.1.100.12 255.255.255.0
!
ip default-gateway 10.1.100.1
!
workgroup-bridge client-vlan 200
workgroup-bridge timeouts eap-timeout 4 <-- sets the maximum time for the EAP auth process to complete
workgroup-bridge timeouts iapp-refresh 100 <-- sets the timer for an IAPP retry to be send after the initial IAPP message
workgroup-bridge timeouts auth-response 800 <-- sets the timers for how long the WGB will wait for a parent to reply to an Auth Req
workgroup-bridge timeouts assoc-response 800 <-- sets the timers for how long the WGB will wait for a parent to reply to an Assoc Req
workgroup-bridge timeouts client-add 800 <-- sets the timers for how long the WGB will wait for a parent to reply to an Client Add Req
!
end
```
