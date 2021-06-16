---
sort: 1
---

# cgroups
cgroups are a way to restrict and manage users and processes.

One use case for cgroups on login nodes is:
* to restrict all users collectively 
    * to 80% of the total CPU, reserving 20% CPU for the system
    * to 90% of the total memory
    * to 80% of the total swap
* to restrict each user individually
    * to no more than 4 CPU cores
    * to no more than 24 GiB
    * to some small value of swap space

A policy like the above will prevent individual users from overwhelming a login node and causing performance degradation for the other users on the same login node.
Policies like this also incentivize use of the job scheduler, as intended.

## cgroups-v2 on EL8 systems
To enact a policy like this for an EL8 system (e.g. RHEL 8) running systemd with cgroups-v2 support, first create the file `/etc/systemd/system/user.slice` with the following contents to limit all users collectively:

```systemd
#  SPDX-License-Identifier: LGPL-2.1+
#
#  This file is part of systemd.
#
#  systemd is free software; you can redistribute it and/or modify it
#  under the terms of the GNU Lesser General Public License as published by
#  the Free Software Foundation; either version 2.1 of the License, or
#  (at your option) any later version.

[Unit]
Description=User and Session Slice
Documentation=man:systemd.special(7)
Before=slices.target

[Slice]
CPUQuota=2560%
MemoryHigh=80%
MemoryMax=90%
MemorySwapMax=80%
```

```note
The 2560% here is assuming a 32-core login node.

`32 cores * 100% * 0.8 = 2560%`
```

Second, to limit users individually, create the file `/etc/systemd/system/user-.slice.d/10-defaults.conf` with the following contents:

```systemd
#  SPDX-License-Identifier: LGPL-2.1+
#
#  This file is part of systemd.
#
#  systemd is free software; you can redistribute it and/or modify it
#  under the terms of the GNU Lesser General Public License as published by
#  the Free Software Foundation; either version 2.1 of the License, or
#  (at your option) any later version.

[Unit]
Description=User Slice of UID %j
After=systemd-user-sessions.service

[Slice]
TasksMax=80%
CPUQuota=400%
MemoryHigh=20G
MemoryMax=24G
MemorySwapMax=0G
```

```note
These files require cgroups-v2 (the unified hierarchy) to be enabled.

You can enable cgroups-v2 by adding the following option to the kernel command line:
`systemd.unified_cgroup_hierarchy=1`
```

## References

1. [systemd resource control man page](https://www.freedesktop.org/software/systemd/man/systemd.resource-control.html)
