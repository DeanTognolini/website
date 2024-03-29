---
title: "Podman Networking"

date: 2022-12-11
url: /podman-networking/
image: images/2022-thumbs/podman-basics.png
categories:
  - Linux
  - Networking
tags:
  - Containers
draft: false
-----

# Podman Network Types
There are three main networking types in Podman: bridge, macvlan, and slirp4netns.

## bridge
The bridge network type is the default.

Containers can initiate communications to hosts outside of the host system, i.e. the internet via the host systems network interface.

For inbound communications the traffic is addressed to the host systems IP address and a given port number that maps to a particular container. If the host systems firewall permits, the host forwards the incomming traffic to the container based on the configured port mappings.

Podman creates a bridge network by default, you can see this network with the `podman network ls` command, and `podman network inspect <network_name>` for more details.

![podman-networking-bridge](/images/2022/podman-networking-bridge.png)

## macvlan
macvlan allows a container access to a physical network interface on the host, which can be configured with multiple sub-interfaces with its own IP and MAC address. This allows the container to appear as if it is directly connected to the same network as the host system.

Outside hosts will be able to communicate directly with the containers IP address.

![podman-networking-macvlan](/images/2022/podman-networking-macvlan.png)

## slirp4netns
slirp4netns is similar to a bridge network in the sense that containers can communicate with outside hosts. slirp4netns is used by containers built by unprivileged users that are not allow to create network interfaces on the host system. Containers using slirp4netns are isolated and cannot communicate with other containers on the system.

Containers must use ports 1024 through 65535 as ports below 1024 need root privileges to provide access.
