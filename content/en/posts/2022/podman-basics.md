---
title: "Podman Basics"

date: 2022-12-10
url: /podman-basics/
image: images/2022-thumbs/podman-basics.png
categories:
  - Linux
tags:
  - Containers
draft: false
-----

# Installing Podman
If you need to install Podman from scratch see https://podman.io/getting-started/installation.

My host system is running Alma Linux with `podman`, `cockpit` and `cockpit-podman` pre-installed, you can enable Podman directly from the Cockpit webGUI in the "Podman containers" menu. If you can't see this you may need to install the `cockpit-podman` package.

![podman](/images/2022/podman.png)


# Finding and Pulling Images
`$ podman search <search_term>` to search for images. You can also use images from Docker Hub or equivalent.
`$ podman pull <image_name>` to pull images locally
`$ podman images` lists all locally pulled images

# Running a Container
`$ podman run -dt --name httpd -p 8080:80/tcp docker.io/library/httpd` runs the container with some options:
* -d enabled detached mode
* -t enables an interactive TTY to issue commands in the container enviornment
* --name gives the container a friendly name so we don't need to use the generated random ID string to refernce the container.
* -p specifics the network ports to be mapped.

`$ podman ps` lists running containers on the system
`$ podman ps -a` lists all containers on the system

# Managing a Container
`$ podman inspect <container_name> | grep ` lists the details of the container. You can use grep to filter the output.
`$ podman logs <container_name>` shows the container logs.
`$ podman stop <container_name>` stops the container.
`$ podman restart <container_name>` restarts the container.
`$ podman rm <container_name>` remove the container from the system.
`$ podman rmi <image_id>` remove the image from the system.
`$ podman top <container_name>` lists the running processes in the container.
`$ podman stats` starts a live displayer of container resource usage.

