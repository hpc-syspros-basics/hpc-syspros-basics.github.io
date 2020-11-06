---
sort: 2
---

# Configuring NHC

After installing NHC we can start to put in some configuration to have the script start verifing the health of nodes. The starting configuration is meant to highlight the range of checks that you could have NHC check to ensure a node is healthy before it will run jobs.

When configuring your `nhc.conf` file you should keep in mind that checks are executed from the top of the file down. Therefore you may want to place the checks that you care about most at the top of the file, and less important checks near the bottom of the configuration file. Generally hardware checks should come first followed by checks for software, BIOS configuration, filesystems, and process checks. To illustrate this we can look at the following configuration:

```text
# NHC Configuration file
# https://github.com/mej/nhc

* || export NHC_RM=slurm
* || export TIMEOUT=15

* || check_reboot_slurm
* || check_gpfs_remediation
* || check_remediation

# Hardware checks
shas* || check_hw_cpuinfo 2 24 24
ssky* || check_hw_cpuinfo 2 24 24
smem* || check_hw_cpuinfo 4 48 48
sknl* || check_hw_cpuinfo 1 68 272
/^sgpu0[1-5][01-02]/ || check_hw_cpuinfo 2 24 24
sgpu0801 || check_hw_cpuinfo 2 28 28

/^s(has|gpu)/ || check_hw_physmem 128gb 128gb 5%
ssky* || check_hw_physmem 192gb 192gb 5%
smem* || check_hw_physmem 2tb 2tb 5%
sknl* || check_hw_physmem 96gb 128gb 5%

# Ensure our firmware on the HFI is up to date
* || check_cmd_output -t 10 -m 'Current Firmware Version=10.9.0.0.208' -e 'opatmmtool'

# OPA RDMA acceleration
* || check_file_contents /sys/module/hfi1/parameters/cap_mask 0x4c09a01cbba

# Ensure that NTP is synchronized
* || check_cmd_output -t 2 -m 'NTP synchronized: yes' -e 'timedatectl'

# Ensure that mmsysmon is stopped on the node
* || check_ps_service -E '/usr/lpp/mmfs/bin/mmsysmoncontrol stop' -f -d 'mmsysmon.py' mmsysmon

# GPFS service and mount checks, we additionally check for a file to find nodes with stale mounts
* || check_ps_service -d mmfsd gpfs-summit.mount
* || check_fs_mount_rw -f /gpfs/summit
* || check_file_test -r -e -f /scratch/summit/.sentinel

# Check that the nodes have the specified BIOS version
shas* || check_dmi_data_match "BIOS Information: Version: 2.11.0"
ssky* || check_dmi_data_match "BIOS Information: Version: 1.4.9"
sgpu* || check_dmi_data_match "BIOS Information: Version: 2.11.0"
smem* || check_dmi_data_match "BIOS Information: Version: 2.8.1"
sknl* || check_dmi_data_match "BIOS Information: Version: 2.3.0"

# Check that LDAP is resolving correctly on the node
* || check_cmd_status -t 5 -r 0 id rcops

# Check MCElog for any uncorrectable errors in the last 24h
* || check_hw_mcelog

# Check Infiniband and management ethernet connections
* || check_hw_ib 100
/^s(has|gpu|mem)/ || check_hw_eth eno1
sknl* || check_hw_eth enp4s0
ssky* || check_hw_eth eno16

# Filesystem checks
* || check_fs_mount_rw -f /
* || check_fs_free /dev/mapper/vg_root-lv_root 5%
* || check_fs_ifree /dev/mapper/vg_root-lv_root 1k

* || check_fs_free /var 5%
* || check_fs_ifree /var 1k

* || check_fs_mount_rw -f /tmp
* || check_fs_free /tmp 1%
* || check_fs_ifree /tmp 100

* || check_fs_free /dev/shm 25%

* || check_fs_mount_rw -f /beegfs/pl-active
* || check_file_test -r -e -f /beegfs/pl-active/.sentinel

* || check_fs_mount_rw -f /scratch/local
* || check_fs_free /scratch/local 1%
* || check_fs_ifree /scratch/local 100

* || check_fs_mount_rw -t "nfs" -s "isilon1:/ifs/curc/projects" -F "/projects"
* || check_fs_mount_rw -t "nfs" -s "isilon1:/ifs/curc/home" -F "/home"

# Check that sshd is owned by root
* || check_ps_service -u root -S sshd

# Check for rogue processes, if found log out to the NHC log and system log
* || check_ps_userproc_lineage log syslog
```
In this example we start with some environment variables that are setup. Notably that the resource manager we are using is Slurm, and that we have set a Timemout value of 15 seconds, meaning if NHC can't evaluate all checks within 15 seconds, the node will be marked down until NHC can run through all the checks.

We then go through some hardware checks, particulalry for our CPUs on the machines.
```text
shas* || check_hw_cpuinfo 2 24 24
ssky* || check_hw_cpuinfo 2 24 24
smem* || check_hw_cpuinfo 4 48 48
sknl* || check_hw_cpuinfo 1 68 272
/^sgpu0[1-5][01-02]/ || check_hw_cpuinfo 2 24 24
sgpu0801 || check_hw_cpuinfo 2 28 28
```
As you might notice we have a few different types of nodes here each with varying processor counts and core counts. For most nodes the CPUs have only 12 cores, except a newer GPU node as well as all of our Knight's Landing Nodes which have 68 cores on each CPU, and 4 threads per core for a total of 272 threads.

After checking CPUs we check how much memory is still left on the node
```text
/^s(has|gpu)/ || check_hw_physmem 128gb 128gb 5%
ssky* || check_hw_physmem 192gb 192gb 5%
smem* || check_hw_physmem 2tb 2tb 5%
sknl* || check_hw_physmem 96gb 128gb 5%
```
So in our hardware memory checks we want to ensure that on a node there is at least a certain amount of memory on a node, and we give it a fugde factor of 5%, meaning the hardware value could be within 5% of the values specified and the check would still pass. This is helpful to avoid getting bitten by any changes in the way the kernel reports out memory. You should make the fudge factor large enough to avoid any errors after upgrading the kernel or BIOS on a machine, but still small enough to pick up any missing memory DIMMs.

We also have a few command and service checks as well
```text
# Ensure our firmware on the HFI is up to date
* || check_cmd_output -t 10 -m 'Current Firmware Version=10.9.0.0.208' -e 'opatmmtool'
  
# OPA RDMA acceleration
* || check_file_contents /sys/module/hfi1/parameters/cap_mask 0x4c09a01cbba
  
# Ensure that NTP is synchronized
* || check_cmd_output -t 2 -m 'NTP synchronized: yes' -e 'timedatectl'

# GPFS service and mount checks, we additionally check for a file to find nodes with stale mounts
* || check_ps_service -d mmfsd gpfs-summit.mount
* || check_fs_mount_rw -f /gpfs/summit
* || check_file_test -r -e -f /scratch/summit/.sentinel
```
So we check that the Omnipath Firmware version on the node Host Fabric Interface is at a certain version. Most importantly we check to ensure that NTP is synchronized on a node by querying for a string out of the output of timedatectl to ensure that clocks are in sync across the cluster. We also have two service checks, one to ensure that the GPFS monitoring daemon is not running on the node, and another check to ensure that our gpfs mounting service is running to provide fast scratch storage on the node.

Additionally we also check that GPFS is mounted at `/gpfs/summit` and is available as a read-write mount. We then ensure that we can access the contents of the GPFS mount by ensuring that we can reach a sentinel file in the GPFS filesystem to test for stale mounts.

```text
# Check that the nodes have the specified BIOS version
shas* || check_dmi_data_match "BIOS Information: Version: 2.11.0"
ssky* || check_dmi_data_match "BIOS Information: Version: 1.4.9"
sgpu* || check_dmi_data_match "BIOS Information: Version: 2.11.0"
smem* || check_dmi_data_match "BIOS Information: Version: 2.8.1"
sknl* || check_dmi_data_match "BIOS Information: Version: 2.3.0"
```

In the section of the configuration above we check for the BIOS version that is installed on a node by using the dmi data match check which queries the results of dmidecode in order to verify the installed BIOS version on the motherboard.

```text
# Check that LDAP is resolving correctly on the node
* || check_cmd_status -t 5 -r 0 id rcops

# Check MCElog for any uncorrectable errors in the last 24h
* || check_hw_mcelog
```

We then check a few commands and their output, in the first check we ensure that the node is able to resolve a username that is setup in LDAP to confirm the node is able to pull user information from centralized LDAP servers. We then check that `mcelog` has not detected any uncorrectable errors in the last 24 hours.

```text
# Check Infiniband and management ethernet connections
* || check_hw_ib 100
/^s(has|gpu|mem)/ || check_hw_eth eno1
sknl* || check_hw_eth enp4s0
ssky* || check_hw_eth eno16
```

In this section we ensure that each node has the proper network interfaces configured that we care about. We ensure that all devices have a Infiniband or Omnipath interface setup and that it has a data rate of 100Gb/s. Finally we check to ensure that we have a management ethernet interface available.

```text
# Filesystem checks
* || check_fs_mount_rw -f /
* || check_fs_free /dev/mapper/vg_root-lv_root 5%
* || check_fs_ifree /dev/mapper/vg_root-lv_root 1k

* || check_fs_free /var 5%
* || check_fs_ifree /var 1k

* || check_fs_mount_rw -f /tmp
* || check_fs_free /tmp 1%  
* || check_fs_ifree /tmp 100

* || check_fs_free /dev/shm 25%

* || check_fs_mount_rw -f /beegfs/pl-active
* || check_file_test -r -e -f /beegfs/pl-active/.sentinel

* || check_fs_mount_rw -f /scratch/local
* || check_fs_free /scratch/local 1%
* || check_fs_ifree /scratch/local 100

* || check_fs_mount_rw -t "nfs" -s "isilon1:/ifs/curc/projects" -F "/projects"
* || check_fs_mount_rw -t "nfs" -s "isilon1:/ifs/curc/home" -F "/home"
```

Here we have a section full of filesystem checks to ensure that various filesystems on the system are mounted, and that they have a certain percentage of space free, and that there are enough available inodes. Additionally we have some checks for various filesystems to ensure that they are of the correct mount type and that they are mounted at the correct locations and that they can be written to.

```text
# Check that sshd is owned by root
* || check_ps_service -u root -S sshd

# Check for rogue processes, if found log out to the NHC log and system log
* || check_ps_userproc_lineage log syslog
```

Finally we have a final process check that ensures that the ssh server daemon is owned by the root user, and that any rogue processes, or processes owned by a user that has not been authorized by the resource manager to run on the node is logged out to the NHC log file and additionally logged out to the system log.

This is just a small example of what is possible with the NHC checks, and additionally you can write your own checks as well to fit any edge cases you may encounter with various pieces of hardware or processes and services you may be running at your own site. You can read more about the various other built-in checks on the NHC github documentation pages included as references at the bottom of this article.

---
## References

1. [NHC Documentation](https://github.com/mej/nhc/blob/master/README.md)
2. [NHC Configuration](https://github.com/mej/nhc#configuration)
3. [NHC Built-in Checks](https://github.com/mej/nhc#built-in-checks)
