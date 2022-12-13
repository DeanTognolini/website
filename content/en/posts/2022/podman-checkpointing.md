---
title: "Podman Checkpointing"

date: 2022-12-13
url: /podman-checkpointing/
image: images/2022-thumbs/podman-basics.png
categories:
  - Linux
tags:
  - Containers
  - Podman
draft: false
-----
# What is Checkpointing?
Checkpointing stops the container and freezes it in the current running state that can be restored at a later point in time.

Checkpointing currently only works with rootful containers.

Checkpointing requires the `criu` package to be installed, you can read more about CRIU [here](https://criu.org/Main_Page). 

# Checkpointing a Container
The `# podman container checkpoint <container_id>` command is use to checkpoint a container. This will freeze the container in the current running state and stop the container.

# Restoring a Container
The `# podman container restore <container_id>` command is used to restore a container to the checkpoint. The container will be restored from the exact point in time it was checkpointed.

# Migrating a Container
The `# podman container checkpoint <container_id> -e /tmp/mycheckpoint.tar.gz` will save a compressed version of the container. This file can then be transferred to another system using a method such as `scp` and restored with the `# podman container restore -i /tmp/mycheckpoint.tar.gz` command.
