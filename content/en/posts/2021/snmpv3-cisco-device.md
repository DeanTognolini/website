---
title: "SNMPv3 on a Cisco Device"

date: 2021-08-09
url: /snmpv3-cisco-device/
image: images/2021-thumbs/snmpv3-cisco-device.webp
categories:
  - Networking
tags:
  - Cisco
  - SNMPv3
draft: false
-----
![snmpv3-cisco-device.webp](/images/2021/snmpv3-cisco-device.webp)

Simple Network Management Protocol version 3 introduces much needed security features to the SNMP protocol, SNMPv1 and v2c do not support authentication or encryption of SNMP data. SNMPv3 introduces authentication based on the HMAC-MD5 and HMAC-SHA algorithms, and encryption using AES algorithms (weaker algorithms are also supported).

SNMPv3 does away with the legacy concept of community strings, and instead uses groups and users to control access. Each user can be configured with different credentials and assigned to different groups. Groups can be configured with different settings such as ACLs.

SNMPv3 supports three levels of security:
1. noAuthNoPriv
2. AuthNoPriv
3. AuthPriv

noAuthNoPriv means no authentication or encryption is used. AuthNoPriv means authentication is used but not encryption. AuthPriv means both authentication and encryption is used.

# Configuring noAuthNoPriv
The below shows a SNMP user called XOGS being created and added to the XOGS_SNMPv3_GROUP using SNMPv3. No authentication key or encryption key has been configured for the user. We can also see the XOGS_SNMPv3_GROUP is configured to use security model **v3 noauth**.

```>:
R1(config)#snmp-server user XOGS XOGS_SNMPv3_GROUP v3
R1(config)#do sh snmp user
User name: XOGS
Engine ID: 800000090300500000010000
storage-type: nonvolatile        active
Authentication Protocol: None
Privacy Protocol: None
Group-name: XOGS_SNMPv3_GROUP
R1(config)#snmp-server group XOGS_SNMPv3_GROUP v3 noauth
R1(config)#do sh snmp group
groupname: XOGS_SNMPv3_GROUP                security model:v3 noauth 
contextname: <no context specified>         storage-type: nonvolatile
readview : v1default                        writeview: <no writeview specified>        
notifyview: <no notifyview specified>       
row status: active
```

# Verifying noAuthNoPriv
On the Solarwinds Orion VM I have connected to R1 I will use SNMPWalk to test the SNMP configuration. Sending some SNMP GET messages to R1 will succeed because we have not enabled any SNMP security features on R1.

![snmpv3-cisco-device2.webp](/images/2021/snmpv3-cisco-device2.webp)

In the below pcap we can see that the SNMP GET message from 192.168.135.10 (Solarwinds Orion VM) has been successful and R1 has returned an SNMP report.

![snmpv3-cisco-device2.webp](/images/2021/snmpv3-cisco-device2.webp)

# Configuring AuthNoPriv
The below shows an SNMP user called XOGS being created and added to the XOGS_SNMPv3_GROUP using SNMPv3. A authentication key has been configured but no encryption key has been configured. We can also see the XOGS_SNMPv3_GROUP is configured to use security model **v3 auth**.

```>:
R1(config)#snmp-server user XOGS XOGS_SNMPv3_GROUP v3 auth sha auth123
R1(config)#do sh snmp user
User name: XOGS
Engine ID: 800000090300500000010000
storage-type: nonvolatile        active
Authentication Protocol: SHA
Privacy Protocol: None
Group-name: XOGS_SNMPv3_GROUP
R1(config)#snmp-server group XOGS_SNMPv3_GROUP v3 auth
R1(config)#do sh snmp group   
                         
groupname: XOGS_SNMPv3_GROUP                security model:v3 auth 
contextname: <no context specified>         storage-type: nonvolatile
readview : v1default                        writeview: <no writeview specified>        
notifyview: <no notifyview specified>       
row status: active
```

# Verifying AuthNoPriv
Now that we have enable SNMPv3 authentication, we must ensure our SNMP GET messages are sent with correct authentication credentials. I have configured SNMPWalk to use authentication with username XOGS and password auth123 as configured on R1.

![snmpv3-cisco-device3.webp](/images/2021/snmpv3-cisco-device3.webp)

In the below pcap we can see the SNMP GET message was sent with the Authenticated bit set, and authentication parameters further down.

![snmpv3-cisco-device4.webp](/images/2021/snmpv3-cisco-device4.webp)

# Configuring AuthPriv
The below shows a SNMP user called XOGS being created and added to the XOGS_SNMPv3_GROUP using SNMPv3. Both an authentication key and encryption key has been configured for the user. We can also see the XOGS_SNMPv3_GROUP is configured to use security model **v3 priv**.

```>:
R1(config)#snmp-server user XOGS v3 auth sha auth123 priv aes 256 priv123
R1(config)#do sh snmp user 
User name: XOGS
Engine ID: 800000090300500000010000
storage-type: nonvolatile        active
Authentication Protocol: SHA
Privacy Protocol: AES256
Group-name: XOGS_SNMPv3_GROUP
R1(config)#snmp-server group XOGS_SNMPv3_GROUP v3 priv 
R1(config)#do sh snmp group
groupname: XOGS_SNMPv3_GROUP                security model:v3 priv 
contextname: <no context specified>         storage-type: nonvolatile
readview : v1default                        writeview: <no writeview specified>        
notifyview: <no notifyview specified>       
row status: active
```

# Verifying AuthPriv
Now that we have enable SNMPv3 authentication and encryption, we must ensure our SNMP GET messages are sent with correct authentication and privacy credentials. I have configured SNMPWalk to use authentication with username **XOGS** and password **auth123**, and privacy with password **priv123** as configured on R1.

![snmpv3-cisco-device5.webp](/images/2021/snmpv3-cisco-device5.webp)

In the below pcap we can see the Authentication bit is set, the PDU has been encrypted and is now unreadable to wireshark.

![snmpv3-cisco-device6.webp](/images/2021/snmpv3-cisco-device6.webp)
