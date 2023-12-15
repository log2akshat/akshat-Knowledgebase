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

