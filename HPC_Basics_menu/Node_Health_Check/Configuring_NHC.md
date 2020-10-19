---
sort: 2
---

# Configuring NHC

After installing NHC we can start to put in some configuration. The starting configuration is meant to highlight the range of checks that you could have NHC check to ensure a node is in a healthy state before we start to run jobs.

When configuring out your `nhc.conf` file you should keep in mind that checks are executed from the top of the file down. Therefore you may want to place the checks that you care about most at the top of the file, and lesser important checks near the bottom of the configuration file. Generally hardware checks should come first followed by checks that check software, BIOS configuration, filesystems, and process checks. Let us look at the following configuration

```
# NHC Configuration file
# https://github.com/mej/nhc

* || export NHC_RM=slurm
* || export TIMEOUT=15

* || check_reboot_slurm
* || check_gpfs_remediation
* || check_remediation

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
/^s(has|gpu|mem|knl)/ || check_cmd_output -t 10 -m 'Current Firmware Version=10.9.0.0.208' -e 'opatmmtool'

# Ensure that NTP is synchronized
* || check_cmd_output -t 2 -m 'NTP synchronized: yes' -e 'timedatectl'

# Ensure that mmsysmon is stopped on the node
/^s(has|sky|gpu|mem|knl)/ || check_ps_service -E '/usr/lpp/mmfs/bin/mmsysmoncontrol stop' -f -d 'mmsysmon.py' mmsysmon

/^s(has|sky|gpu|mem|knl)/ || check_ps_service -d mmfsd gpfs-summit.mount 

/^s(has|sky|gpu|mem|knl)/ || check_hw_ib 100
/^s(has|gpu|mem)/ || check_hw_eth eno1
sknl* || check_hw_eth enp4s0
ssky* || check_hw_eth eno16

shas* || check_dmi_data_match "BIOS Information: Version: 2.11.0"
ssky* || check_dmi_data_match "BIOS Information: Version: 1.4.9"
sgpu* || check_dmi_data_match "BIOS Information: Version: 2.11.0"
smem* || check_dmi_data_match "BIOS Information: Version: 2.8.1"
sknl* || check_dmi_data_match "BIOS Information: Version: 2.3.0"

# OPA RDMA acceleration
#/^s(has|sky|gpu|mem|knl)/ || check_file_contents /sys/module/hfi1/parameters/cap_mask 0x4c09a01cbba

* || check_cmd_status -t 5 -r 0 id rcops

* || check_hw_mcelog

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

/^s(has|sky|gpu|mem|knl)/ || check_fs_mount_rw -f /gpfs/summit
/^s(has|sky|gpu|mem|knl)/ || check_file_test -r -e -f /scratch/summit/.sentinel

* || check_fs_mount_rw -t "nfs" -s "isilon1:/ifs/curc/projects" -F "/projects"
* || check_fs_mount_rw -t "nfs" -s "isilon1:/ifs/curc/home" -F "/home"

* || check_ps_service -u root -S sshd
* || check_ps_userproc_lineage log syslog
```
In this example we start with some environment variables that are setup. Notably that the resource manager we are using is Slurm, and that we have set a Timemout value of 15 seconds, meaning if NHC can't evaluate all checks within 15 seconds, the node will be marked down until NHC can run through all the checks. 

We then go through some hardware checks, particulalry for our CPUs on the machines.
```
shas* || check_hw_cpuinfo 2 24 24
ssky* || check_hw_cpuinfo 2 24 24
smem* || check_hw_cpuinfo 4 48 48
sknl* || check_hw_cpuinfo 1 68 272
/^sgpu0[1-5][01-02]/ || check_hw_cpuinfo 2 24 24
sgpu0801 || check_hw_cpuinfo 2 28 28
```
As you might notice we have a few different types of nodes here each with varying processor counts and core counts. For most nodes the CPUs have only 12 cores, except a newer GPU node as well as all of our Knight's Landing Nodes which have 68 cores on each CPU, and 4 threads per core for a total of 272 threads.

After checking CPUs we check how much memory is still left on the node
```
/^s(has|gpu)/ || check_hw_physmem 128gb 128gb 5%
ssky* || check_hw_physmem 192gb 192gb 5%
smem* || check_hw_physmem 2tb 2tb 5%
sknl* || check_hw_physmem 96gb 128gb 5%
```
So in our hardware memory checks we want to ensure that on a node there is at least a certain amount of memory on a node, and we give it a fugde factor of 5%, meaning the hardware value could be within 5% of the values specified and the check would still pass. This is helpful to avoid getting bitten by any changes in the way the kernel reports out memory. You should make the fudge factor large enough to avoid any errors after upgrading the kernel or BIOS on a machine, but still small enough to pick up any missing memory DIMMs.

We also have a few command and service checks as well
```
# Ensure our firmware on the HFI is up to date
/^s(has|gpu|mem|knl)/ || check_cmd_output -t 10 -m 'Current Firmware Version=10.9.0.0.208' -e 'opatmmtool'

# Ensure that NTP is synchronized
* || check_cmd_output -t 2 -m 'NTP synchronized: yes' -e 'timedatectl'

# Ensure that mmsysmon is stopped on the node
/^s(has|sky|gpu|mem|knl)/ || check_ps_service -E '/usr/lpp/mmfs/bin/mmsysmoncontrol stop' -f -d 'mmsysmon.py' mmsysmon

/^s(has|sky|gpu|mem|knl)/ || check_ps_service -d mmfsd gpfs-summit.mount 
```
So we check that the Omnipath Firmware version on the node Host Fabric Interface is at a certain version. Most importantly we check to ensure that NTP is synchronized on a node by querying for a string out of the output of timedatectl to ensure that clocks are in sync across the cluster. We also have two service checks, one to ensure that the GPFS monitoring daemon is not running on the node, and another check to ensure that our gpfs mounting service is running to provide fast scratch storage on the node.

---
## References

[NHC Documentation](https://github.com/mej/nhc/blob/master/README.md)
