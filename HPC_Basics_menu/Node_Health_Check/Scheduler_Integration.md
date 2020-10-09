---
sort: 4
---

# Scheduler Integration

Commonly NHC is run as part of a resource scheduler at many sites in an effort to catch errors on a node either before, after, or during a job run. The implementation of NHC as part of the resource scheduler varies depending on which scheduler you are using.

## Slurm Integration

Add the following to `/etc/slurm.slurm.conf` (or wherever your `slurm.conf` file is located in your environment) on your controller node(s) AND your compute nodes (because, even though the HealthCheckProgram only runs on the nodes, your slurm.conf file must be the same across your entire system):

```
HealthCheckProgram=/usr/sbin/nhc
HealthCheckInterval=300
```

This will execute NHC every 5 minutes.

## TORQUE/PBS Integration

NHC can be executed by the pbs_mom process at job start, job end, and/or regular intervals (irrespective of whether or not the node is running job(s)). More detailed information on how to configure the pbs_mom health check can be found in the [TORQUE Documentation](http://docs.adaptivecomputing.com/torque/6-1-2/adminGuide/torque.htm#topics/torque/12-troubleshooting/computeNodeHealthCheck.htm) For example:

```
$node_check_script /usr/sbin/nhc
$node_check_interval 5,jobstart,jobend
$down_on_error 1
```

This causes pbs_mom to launch `/usr/sbin/nhc` every 5 "MOM intervals" (45 seconds by default), when starting a job, and when a job completes (or is terminated). Failures will cause the node to be marked as "down."

```note
Some concern has been expressed over the possibility for "OS jitter" caused by NHC. NHC was designed to keep jitter to an absolute minimum, and the implementation goes to extreme lengths to reduce and eliminate as many potential causes of jitter as possible. No significant jitter has been experienced so far (and similar checks at similar intervals are used on extremely jitter-sensitive systems); however, increase the interval to 80 instead of 5 for once-hourly checks if you suspect NHC-generated jitter to be an issue for your system. Alternatively, some sites have configured NHC to detect running jobs and simply exit (or run fewer checks); that works too!
```

In addition, NHC will by default mark the node "offline" (i.e., pbsnodes -o) and add a note (viewable with pbsnodes -ln) specifying the failure. Once the failure has been corrected and NHC completes successfully, it will remove the note it set and clear the "offline" status from the node. In order for this to work, however, each node must have "operator" access to the TORQUE daemon. Unfortunately, the support for wildcards in pbs_server attributes is limited to replacing the host, subdomain, and/or domain portions with asterisks, so for most setups this will likely require omitting the entire hostname section. The following has been tested and is known to work:

```
qmgr -c "set server operators += root@*"
```

This functionality is not strictly required, but it makes determining the reason nodes are marked down significantly easier!

Another possible caveat to this functionality is that it only works if the canonical hostname (as returned by the hostname command or the file /proc/sys/kernel/hostname) of each node matches its identity within TORQUE. If your site uses FQDNs on compute nodes but has them listed in TORQUE using the short versions, you will need to add something like this to the top of your NHC configuration file:

```
* || HOSTNAME="$HOSTNAME_S"
```

This will cause the offline/online helpers to use the shorter hostname when invoking pbsnodes. This will NOT, however, change how the hostnames are matched in the NHC configuration, so you'll still need to use FQDN matching there.
