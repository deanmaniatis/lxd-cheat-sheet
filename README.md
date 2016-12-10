# LXD cheat-sheet

## Installation

On ubuntu:

```
...

```

On RedHat based systems you may have to build lxc/lxd your self. The following example is indicative of the build process for centos7, small variations may exist between centos/redhat/fedora.

#### Build process
```
# enable epel repo if not there already
$ yum -y install epel-release

$ yum install -y go lxc-devel git make dnsmasq squashfs-tools bridge-utils
$ export GOPATH=$HOME/go
$ mkdir $GOPATH
$ go get -v -u github.com/lxc/lxd
$ cd $GOPATH/src/github.com/lxc/lxd

# Check out a stable version
# git checkout 2.0.8
$ make


```

#### Starting the LXD daemon

```
$GOPATH/bin/lxd --group wheel --verbose
```

#### Setup environment for the LXD client

```
$ export GOPATH=$HOME/go
$ export PATH=$PATH:$GOPATH/bin
$ lxc version
```


## Setup DNS for LXD containers (optional)

Create a new configuration file `/etc/NetworkManager/dnsmasq.d/lxd.conf` and append the following:

```
# replace with the IP address configured during "lxd init" stage e.g. 10.0.3.1
server=/lxd/<IP ADDRESS>
```

Restart `NetworkManager` service e.g.

```
$ systemctl restart NetworkManager
```

You can now resolve containers by name e.g.

```
$ lxc list
+---------------+---------+-------------------+------+------------+-----------+
|     NAME      |  STATE  |       IPV4        | IPV6 |    TYPE    | SNAPSHOTS |
+---------------+---------+-------------------+------+------------+-----------+
| c1            | RUNNING | 10.0.3.221 (eth0) |      | PERSISTENT | 0         |
+---------------+---------+-------------------+------+------------+-----------+
| charmbox      | STOPPED |                   |      | PERSISTENT | 1         |
+---------------+---------+-------------------+------+------------+-----------+
| juju-875e27-0 | RUNNING | 10.0.3.220 (eth0) |      | PERSISTENT | 0         |
+---------------+---------+-------------------+------+------------+-----------+

# remember to append .lxd on container's name
$ ping c1.lxd
PING c1.lxd (10.0.3.221) 56(84) bytes of data.
64 bytes from 10.0.3.221: icmp_seq=1 ttl=64 time=0.032 ms
^C
```

## Advanced topics

### Container provisioning
* shell scripting e.g. `bash`
* `cloud-init`
* `ansible`
