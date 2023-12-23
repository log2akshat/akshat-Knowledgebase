# What are Containers ?
Containers are a set of one or more processes that are isolated from the rest of the system. They provide many of the same benefits as Virtual Machines but have many advantages on top of them.

```process == applications```

## History
Containers existed for decades as RedHatters says Containers are Linux it got recently popular in the last 7-8 years as most of the features are implemented needing the containers to be isolated. Many years ago people realized we needed to find a way to isolate the processes and this gave rise to Containers.

## Features in Linux that helps containers to exist
### 1. Control groups (cgroups)
- Control Groups organize all processes in the Linux system, used for Tracking, Grouping, and Organising the processes that run.
- Every process is tracked with cgroups regardless of whether its a container or not. cgroups are typically used to associate the process with resources.
- With a cgroup, we can track how much a group of processes are using for a given kind of resource.
- cgroups also give the ability to Limit or Prioritize what resources are available for a group of processes.

We interact with the cgroups through Subsystem. The Control Group system is an abstract framework while Subsystem is the concrete implementation that is bound to resources. There are a bunch of different subsystems available in the cgroups.

Examples of Subsystem:
- Memory
- CPU Time
- Block I/O
- Number of discrete processes (pid)
- CPU and memory pinning
- Freezer (used by docker pause)
- Devices
- Network priority

cgroups limit the resources that containers can consume so preventing containers from running for a while and exhausting all the system resources or resources of other containers.

Control groups partition sets of processes and their children into groups to manage and limit the resources they consume.

All of them are listed here and most of the susbsystems that are enabled on our system are Resource Controllers means they can Track or Limit a particular type of resource for the processes assigned to the group. One of the important things here is each of the subsystem is independent, they can organize processes separately from each other. This means we can have single CPU cgroups with 2 processes in it. But those same 2 processes are assigned to 2 different memory cgroups.

All of the cgroups subsystem arrange processes in a hierarchy.
- Each subsystem has an independent hierarchy.
- Every task or pid running on the host system is represented in one of the cgroup within a given subsystem hierarchy.
- Independent hierarchy allows more expressive segmentation across resource types. This allows how 2 processes share the total amount of memory they consume gives one process more CPU time than the other. 
- Some Resource controllers apply settings from the parent level to the child levels and others consider each level in the hierarchy independently, for example, the device controller uses this type of inheritance while the memory controller can be configured in either array. For the new processes started it began in the same cgroups as their parent processes.

- We can interact with cgroups with a virtual filesystem typically mounted at /sys/fs/cgroup - a bunch of file and directories appears here but they are just interfaces in the Kernel's data structures for cgroups.
-- Each directory inside the given subsystem root represents a cgroup. In each directory we see a file called tasks, this file holds all of the process IDs for the processes assigned to that particular cgroup. Other files have settings and utilization data or change the settings. To move the process to a different cgroup we just need to write the process id to the target's cgroups tasks file.


## What are Namespaces ?
Namespaces are other Linux Kernel Primitives. Namespaces are concerned with the visibility of Resources and access control. Namespaces can make it appear for the processes that it has its own copy of the resources. In other cases Namespaces can map the resources outside the Namespace to a resource inside a Namespace. Properties like Names or Permissions when a process make changes to a resource within the Namespace generally that change will be within that Namespace not processes outside. The processes effectively have its own copy.

Linux is a bunch of Namespaces that covers different kind of resources, like in recent versions of Linux new Namespaces are being developened as well.
- Network
- Filesystem (mounts)
- Processes (pid)
- Inter-process communication (ipc)
- Hostname and domain name (uts)
- User and group IDs
- cgroup

Just like cgroups, processes can be in any combination of Namespaces. Processes can share different Namespaces with each other. For example we could have mount Namespace that is shared between most processes who want to run one of them it in a separate network or pid Namespace. Or we can run a process with its own Namespacees much like a container.


# Namespaces frequently used in a Container:
--------------------------------------------
- Network Namespace
-------------------
Network Namespace is a seprated view of the network with different network interfaces enviornment role. Network Namespaces can be connected together with a virtual ethernet device pair or e-eth pair. Docker typically uses a separate network Namespace for container and by default configures each container Namespace such as veth pair connected to a Linux bridge to enable outbound connectivity.

While Kubernetes pods and ECS tasks to get a unified view of the network in the single IP address expose to all of containers in the same Pod or tasks.

Linux containers have the separate view of the filesystem with their own set of files from the container Image. Now, Namespaces are the mechanism for providing the separate view. The container image mounted as a container root filesystem adding the host filesystem from view.

To share data across containers or between the host and the container typically done by using Volumes. Volumes are really a mount.

- procfs virtual filesystem
----------------------------
procfs virtual filesystem is a mechanism that the Linux Kernel exposes for the introspection of its data. This filesystem includes lots of information about each process that is running. One of the piece of information that is available is the Namespaces to which the processes belongs. Inside the process directory there is a directory called `ns`. This directory contains the symbolic link to the Namespace. However, they are not quiet regular symbolic link that points the files. These are the symbolic link structure to note the Namespace type and the inode number that the kernel uses to identify the Namespace. The number itself is not really meaningful but we can use it to understand whether 2 processes belong to the same Namespace or not.

Manipulate Namespaces
---------------------
Unlike cgroups Namespaces are not manipulated through a virtual filesystem, instead a set of Linux syscalls are used for working with the Namespaces. There are 2 syscalls that can be used to create a Namespaces:
-- clone
-- unshare

clone is the syscall for creating the Namespaces as part of its functionality we can start a new process in a new Namespace. The Namespaces which it creates is controlled by a set of flag that start with CLONE_NEW.
unshare is a syscall that a running process can use to move itself in a new Namespace, unshare supports the same flag as clone.

Persisting Namespaces
---------------------
Unlike cgroups Namespaces can be empty, Namespaces automatically close when nothing is holding it open. There are 2 options for occupying Namespaces either a running processes or a `bind` mount. When we have a bind mount then the kernel will maintain the Namespace independently of the running process.


Entering Namespaces
---------------------
For a running process to enter a existing Namespace it can use the `setns` syscalls. This syscall refers a file descriptor to act as a identifier for the target Namespace to enter. The file descriptor can be obtained by obtaining one of the Namespaces links in proc or by opening a bind mount of it elsewhere on the system. Once the process is moved into a Namespace it holds the Namespace open even if the orginal symlink or bind mount goes away. There are couple of tools that helps us to enter the Namespaces without making syscalls ourselves like:
-- `nsenter` - for doing it interactively
--  `ip-netns` - works specifically for network Namespaces
