---
title: "Cisco WGB Bridging Table Issue"

date: 2022-12-15
url: /cisco-wgb-bridge-issue/
image: images/2022-thumbs/cisco-wgb-config1.png
categories:
  - Networking
tags:
  - Cisco
  - Wireless
draft: false
-----
# The Issue
I had an issue on a recent project where I had a WGB-client directly connected to Gi0 on a Cisco IW3702 WGB that I wanted to be in VLAN200 so I configured the `workgroup-bridge client-vlan 200` command, therefore the MAC address for this WGB-client should be in bridge-group 200 (BG200). However the WGB was being strange and putting the WGB-client into both the BG1 and BG200 bridging tables which would break communications as the WGB didn't know where to correctly forward packets intended for this client!

![cisco-wgb-config1.png](/images/2022/cisco-wgb-config1.png)

I am running the latest recommended code (ap3g2-k9w7-tar.153-3.JPM) so I think it might be a bug, I also know that Cisco doesn't recommended multiple VLAN deployments behind a WGB but it is possible and widely used in my experience.

It seems like the WGB doesn't active the client-vlan config until after the interface fires up and that is how the MAC ends up in both bridging tables?

# Work Around
I came up with a workaround by using a MAC ACL to block the WGB-client MAC from being able to communicate on BG1 as below. This works fine but I don’t think this is intended behaviour.

```
!
interface GigabitEthernet0
 bridge-group 1 input-address-list 700
 bridge-group 1 output-address-list 700
!
access-list 700 deny 000a.7529.f32e 0000.0000.0000
!
```
Until I figured out the above solution I was using a Python script to SSH into the WGB and clear the BG1 table.
```Python
#!/usr/bin/env python
from netmiko import Netmiko

#  set device dictionary to pass to net_connect
wgb01 = {
    "host": "10.1.100.12",
    "username": "admin",
    "password": "password",
    "device_type": "cisco_ios",
}

# set list of commands to pass to net_command.send_config_set
commands_wgb01 = [
    "clear bridge 1",
]

#  ssh to device, pass in dictionary to net_connect
net_connect = Netmiko(**wgb01)

print("\nRunning commands to WGB01...\n\n" + net_connect.find_prompt())
output = net_connect.send_config_set(commands_wgb01)
output = net_connect.save_config
print(output)
print("\n\n")
output = net_connect.send_command("sh bridge")
print(output)
print("\n\nCommands to WGB01 complete")
net_connect.disconnect()
```
