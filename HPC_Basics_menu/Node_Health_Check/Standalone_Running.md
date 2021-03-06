---
sort: 3
---

# Standalone Running NHC

Most the time you will want to run NHC as part of your resource scheduler to be performed at the beginning and end of any job on a node, as well as at some determined interval while jobs are
running on the node. There is also value in running NHC as a standalone process to pick up any misconfiguration or missing components for instance when you are checking on a node after hardware
repairs or checking all nodes in a cluster following a maintenance period.

To run NHC as a standalone process to check a node's health you can just run the binary from `/usr/sbin/nhc` as shown below:

```
/usr/sbin/nhc
```
```note
If you installed the NHC binary in another location, you will need to adjust the above command to point to the path where you installed NHC
```

The previous command will execute the node health check script until NHC encounters a check that fails. When the NHC script encounters a error it will print out the check that it failed against
and will imediately exit out.

Having the command exit on the first failure may be desired, but many users may want to find all checks that would fail for a node and be informed about all the errors on a node that need to be
addressed in order make the node healthy again. To check all components as specified in the node health check configuration we can do the following:

```
/usr/sbin/nhc -a
```

The previous command adds the `-a` flag which tells the node health check to not stop at the first check failure, and instead tells it to execute all the checks regardless of if a check fails or
not, giving us a more complete view or what needs to be adjusted or fixed on a node in order to make it healthy again.

---
## References

1. [NHC Documentation](https://github.com/mej/nhc/blob/master/README.md)
