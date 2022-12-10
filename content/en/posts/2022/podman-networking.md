---
title: "Podman Networking"

date: 2022-12-10
url: /podman-networking/
image: images/2022-thumbs/podman-networking.jpg
categories:
  - Linux
  - Networking
tags:
  - Containers
draft: true
-----

# Podman Network Types
There are three main networking types in Podman: bridge, macvlan, and slirp4netns.

## bridge
The bridge network type is the default.

Containers can initiate communications to hosts outside of the host system, i.e. the internet via the host systems network interface.

For inbound communications, that is traffic coming from outside of the host system to a container, the traffic is typically addressed to the host systems IP address and a given port number that maps to a particular container. If the host systems firewall permits, the host knows to forward the incomming traffic to the container based on the configured port mappings.

Podman creates a bridge network by default, you can see this network with the `podman network ls` command, and more details with the `podman network inspect <network_name>` command.
