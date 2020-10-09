---
sort: 1
---

# Installing NHC

The node health check script source can be found at [https://github.com/mej/nhc](https://github.com/mej/nhc) and with each release pre-built RPM packages are distributed along side the source code for easy installation on Redhat based operating systems for versions 4, 5, 6, and 7. If you are running a Redhat based release you can just download the appropriate RPM for your OS major version and install using yum. Otherwise you can install the node health check from source using the following commands from the source code directory after unpacking the tarball.

```
# If installing on a stateful node
yum install lbnl-nhc-1.4.2-1.el7.noarch.rpm

# Or if you are installing into a OS tree for stateless installs
yum --installroot=/stateless/install/location lbnl-nhc-1.4.2-1.el7.noarch.rpm

# If installing from source
tar -zxvf nhc-1.4.2.tar.gz
cd nhc-1.4.2
./configure --prefix=/usr --sysconfdir=/etc --libexecdir=/usr/libexec
make test
make install
```
```note
Executing the make test step in the previous example is optional but recommended as this will run NHC's built-in unit test suite to make sure everything is functioning properly.
```

After following the above installation steps the script will be installed as `/usr/sbin/nhc`, the configuration file and check scripts in `/etc/nhc`, and the helper scripts in `/usr/libexec/nhc`.
