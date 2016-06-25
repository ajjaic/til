# Installing and configuring LXD with ZFS Loop file
* Install zfs with `sudo apt-get install zfsutils-linux`
* Configure lxd with `sudo lxd init`
* Choose `zfs` as the storage backend
* Choose `no` when asked to if you want to use existing block device
* Choose a size for the zfs loop file
* Choose whether you want the lxd daemon to be available over the network. In
  this case, you can choose `no`
* Choose `yes` to configure the lxd bridge.
* Setup an IPv4 subnet by choosing the subnet Gateway, Network mask and DHCP
  range
* And finally choose `Yes` for NAT'ing the traffic
* If you want IPv6 connectivity for your containers you can proceed to setup
  the IPv6 subnet
* Add user to the `lxd` group

# What is LXD
* LXD is a container hypervisor.
* Can run on any host as a daemon
* This daemon manages containers in that host. It can create, destroy, modify
  the container running in that host
* Communication with the daemon requires a client. 2 clients available at the
  moment
  * lxc client (cli)
  * opnstack nova plugin
* LXD containers have many config options for
  * RAM allocation
  * Disk allocation
  * Network allocation
  * CPU allocation
* LXD containers boot a full OS on their file system
* Containers are launched as non-root by default
* In unprivileged containers root inside a container is not root on the host.
  Same for other normal users in the container.
* Communication between LXD and clients is via SSL/TLS.
* All the containers are stored in `/var/lib/lxc`
* A host folder can be shared with multiple containers
* Even container folders can be shared with other containers
* Containers can make use of kernal modules. To allow a container to make use of
  a kernal module, it needs to be first loaded on the host.
* LXD allows device passthrough
* Data can be copied from host to a running container and vice versa just like
  a normal filesystem transfer.

# Container components
A typical container is simply represented by a folder in the host file system.
All the containers are stored in `/var/lib/lxc`.

* rootfs file system
* config options for resource allocations, security and other settings.
* character/block/network device
* a set of profiles from which the container inherits above mentioned config
  options
* properties such as container architecture, persistant or not and name
* runtime state
* If `btrfs` is used as the underlying filesystem, each container is
  automatically created in a `btrfs sub-volume`.

Example if we were to create a container for hosting a wordpress blog then,

* the container is stored at `/var/lib/lxc/wordpress`
* the rootfs of the container is stored at `/var/lib/lxc/wordpress/rootfs`
* the `/var/lib/lxc/wordpress` folder will also contain a `config` and `fstab`
  file
* the `config` file contains the container's configuration
* the `fstab` file is used for mounting host folders into the container.
* the following line in such an `fstab` file,
  ```
  /var/www /var/www bind,create=dir
  ```
  will mount the host `/var/www` into container at `/var/www`
* The same host folder can be mounted in multiple containers to share data

It is possible to mount folders in one container into another container. So it's
possible theoretically for one container to act as a data container with the
rest of the containers mounting directories from the data container. For
simplicity, it is also possible for each container's data to be stored within
itself. Increasing or decreasing the containers size depending on the
requirements.

# LXD Overview
![Container Architecture](file:///home/adas/Documents/til/resources/lxc-vs-baremetal.png)
