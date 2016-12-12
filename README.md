# LXD cheat-sheet

## Installation

On ubuntu:

```
It just works !

```

On RedHat based systems you may have to build lxc/lxd your self. The following example is indicative of the build process for Fedora 24, small variations may exist between centos/redhat/fedora.

#### Build process
```
# enable epel repo if not there already (for Centos 7 only)
yum -y install epel-release

yum install -y go lxc-devel git make dnsmasq squashfs-tools bridge-utils

export GOPATH=$HOME/go
mkdir $GOPATH
go get -v -u github.com/lxc/lxd
cd $GOPATH/src/github.com/lxc/lxd

# Check out a stable version e.g.
# git checkout lxd-2.0.8
make

```

#### Pre-configuration

```
echo "root:1000000:65536" | sudo tee -a /etc/subuid /etc/subgid
```
#### Starting the LXD daemon

```
# Fill usergroup that should have access to lxd
sudo $GOPATH/bin/lxd --group <usergroup for lxd> --verbose
```

#### Setup environment for the LXD client

```
# You may want to add these to your .bashrc/.zshrc
export GOPATH=$HOME/go
export PATH=$PATH:$GOPATH/bin

# You should have now a working lxd daemon and client
lxc version
```

#### Initialize LXD
```
# Use default options
sudo $GOPATH/bin/lxd init
```
#### Misc. other configuration

```
# Check that lxdbr0 bridge exists
ip addr

# If not, follow next steps to manually add a network bridge for your containers.
# You will have to repeat this everytime you restart your machine.
# Consult your distro documentation to make it permanent..
sudo brctl addbr lxdbr0
sudo ip addr add 192.168.12.1 dev lxdbr0

# Do you have linux kernel 4.x ?
uname -r

# If not, upgrade it. Following example is for Centos 7 kernel upgrade to 4.x
# Remember to restart machine
sudo rpm --import https://www.elrepo.org/RPM-GPG-KEY-elrepo.org
sudo rpm -Uvh http://www.elrepo.org/elrepo-release-7.0-2.el7.elrepo.noarch.rpm
sudo yum --enablerepo=elrepo-kernel install kernel-ml
```
### Launch your first LXD container

```
# This will fetch latest ubuntu image from `images` repository and will give it a simple name
lxc launch images:ubuntu/xenial u1

# If the above fails you may have to enable specific kernel options.
```

## Setup DNS for LXD containers (optional)

Create a new configuration file `/etc/NetworkManager/dnsmasq.d/lxd.conf` and append the following:

```
# Replace with the IP address configured during "lxd init" stage e.g. 10.0.3.1
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
