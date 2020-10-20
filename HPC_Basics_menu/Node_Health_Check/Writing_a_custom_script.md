---
sort: 5
---

# Writing a custom check for NHC

Occasionally you will run into an instance where one of the built in checks does not satisfy a use case you have for checking the health of a node. When this happens you have the option of writing and integrating your own check into NHC. Before you begin you should keep in mind that the check should be performant and execute quickly. Additionally you should try to write the check such that it does not call any subshells in keeping with the mantra of NHC. With that in mind lets go ahead and take a look at a custom check example problem.

```tip
It is highly recommended that you consult the [Tips and Best Practices for Checks](https://github.com/mej/nhc#tips-and-best-practices-for-checks) section of the NHC documentation before writing your custom check.
```

## Problem 1: Hardware contexts on Intel QLogic and Omnipath network devices
On our cluster we have an Omnipath network setup as our highspeed network of choice for allowing the nodes to communicate with each other. Our problem is that Omnipath network Host Channel Adapters have objects called contexts which are generated based on how many cores you have on a node. Further you have to ensure that at least one context is available for any jobs to start, otherwise the job will fail. This is a perfect example of a check that is not included by default in NHC since this kind of check is only applicable to Omnipath networks and older Intel QLogic networking devices.

We can start by writing our check script that we will integrate into NHC.

```
# CURC NHC - PSM hardware contexts check

# First instantiate our variable and fill it with the number of free contexts detected on the system
PSM_CONTEXTS=""
PSM_CONTEXTS=$(< /sys/class/infiniband/hfi1_0/nfreectxts)

# Grab the number of free hardware contexts from the HFI device
function check_PSM_contexts() {
  # Setup of variables
  local COUNT="$1"

  if [[ $((PSM_CONTEXTS)) -ge $COUNT ]]; then
    return 0
  fi
  
  # If we receive any return code other than 0 (successful)
  die 1 "$FUNCNAME: PSM hardware contexts exhausted, $PSM_CONTEXTS are available, $COUNT required."
  return 1

}
```
What we see here is that we use the BASH built in read to grab the value out of the system file `/sys/class/infiniband/hfi1_0/nfreectxts`. We then take the input from our check ( this will be the arguement we give the check, an integer value for this check), and compare it against how many hardware contexts are left. If the number of contexts left is still larger than the value we hand the check then the check will succeed and the node will remain online.

If however the number of contexts is smaller than the value we hand the check in our configuration file, then the check will fail and in our case slurm will set the state of the node to DRAIN and set the Reason to "PSM hardware contexts exhausted, $PSM_CONTEXTS are available, $COUNT required".

Once you are done with the script move it to `/etc/nhc/scripts/` so that we can integrate the check into NHC and start calling it. Once it is added to `/etc/nhc/scripts` you can create a configuration entry for it like below.

```
/^s(has|sky|gpu|mem)/ || check_PSM_contexts 24
```
This will now ensure before starting a job that the node has at least 24 contexts available, or that if a node has exhasted the number of hardware contexts that no further jobs start on the node until the hardware contexts free up again.

---
## References

1. [NHC Documentation](https://github.com/mej/nhc/blob/master/README.md)
2. [NHC Tips and Best Practices for Checks](https://github.com/mej/nhc#tips-and-best-practices-for-checks)
