# OMV Luks encryption
You can encrypt your data disks using the LUKS plugin from OMV called
`openmediavault-luksencryption`. It can be installed from the `plugins` tab.

Instead of a password, a key file could be used. The key file enables you to
lock and unlock a LUKS encrypted partition/disk without a password.

A key file can be generated with
```
$ dd bs=512 count=4 if=/dev/urandom of=./mykeyfile iflag=fullblock
```
The above command creates a 2048 byte key. To increase or decrease the size of
the key you can increase or decrease the `count` parameter. A key file can have
a max size of 8192kB.

Sometimes, when the disk is newly encrypted and a new file system is created, it
is possible for the disk to make some click noises when they are mounted. The
reason for those noises could be because the ext4 filesystem lazy creates the
required datastructues on the disk, keeping it quite busy. `iotop` can tell us
which process is writing to the disk so much. For more [info](https://www.reddit.com/r/DataHoarder/comments/3o37gt/external_hdd_clicking_noise_after_encrypting_with/)

# Proxmox Terminal access LXC containers
To access the shell of an LXC container `pct enter <vmid>`
```
pct enter 101
```
To access a login console of an LXC container `pct console <vmid>`
```
pct console 101
```

# Proxmox backups
Before a backup can run, a backup storage must be defined. In most situations,
using a NFS server is the good way to store backups.

The `Max Backups` settings defines the maximum allowed backups of each VM/CT.
That means, a scheduled job will store the defined number of backup of each
VM/CT. If the limit is reached, the oldest backup is removed automatically.

# OMV Installing 3rd party plugins
OMV by default does not come with the plugin that gives access to other
community created plugins. For that we need to manually download and install
the following plugin

 * Visit [omv-extras](http://omv-extras.org/joomla/index.php/guides)
 * Download the 3rd party plugin
 * Upload to OMV and install it
 * Now refresh the set of available plugins

# OMV Storage Pooling
* Install a plugin for storage pooling like the unionfilesystems plugin.
* Choose aufs
* Select pmfs as the creating strategy so you can keep music on one disk and I
  have my movies on two disks. I don't want them scattered about should I want
  to split them apart later.

# OMV Creating NFS shares
* NFS does not do any username/password based authentication.
* It merely allows to mount a file system hosted over the network
* Access control must be provided by the individual applications using the
  share
* Therefore in OMV, it is better to allow **read/write** permissions for
  everyone when using NAS
* Specify restrictions based on IP using the CIDR convention (eg: 10.0.0.0/24)
* If you want to disable IP based restriction to access NFS, then choose `*`
* Use default option of `no_subtree_check` and `secure`
* If using AUFS, then the `/etc/exports/` file in the NFS server should have
  distinct `fsid` options for all the shares. (eg: fsid=1 or fsid=2). It cannot
  be `fsid=0` and must be less than 255
* After editing `/etc/exports/`, to reload the configuration you can use
  `exportfs -r` on the NFS server
* To mount a NFS share in OMV on the client you can use `sudo mount <ip of OMV server>:/export/<sharename> <localpath to mount to>`
  * `(ex: sudo mount 10.0.0.15:/export/proxmox /mnt/nfs)`
  * make sure `nfs-common` is installed
* To unmount the share from the client, try `sudo umount /mnt/nfs`

# Disk Passthrough for KVM VMs
* First identify the disks you want to passthrough with `ls -l /dev/disk/by-id`
* Then allow the disk to passthrough to the VM with
  * `qm set  592  -virtio2 /dev/disk/by-id/ata-ST3000DM001-1CH166_Z1F41BLC`
  * **592** is the VM id.
  * **virtio** is the interface to the disk from the VM. This could also be
    **sata** or **ide** or **scsi**
  * The number at the end of **virtio** is the volume number or device number
    and the range depends on the interface chosen.
  * The last argument is the disk identified by its ID that should be passed
    through to the VM
* To verify that the disks have been added, grep the VM file for the specific
  disk id.
  * Ex: `grep Z1F41BLC /etc/pve/qemu-server/592.conf`

# Proxmox Disk Entities
Proxmox represents various resources related to virtualization using,

* Images - KMV hard disk images
* ISO - KVM installation disk
* Containers - LXC hard disk image
* Templates - LXC installation template/disk
* Backups - Used for taking dumps of KVM images and LXC containers

# Proxmox command line tools
Everything that can be done on the web GUI can be done in CLI as well. Proxmox
has commands for KVMs, LXC, performing backups, managing clusters, storage,
networking and various other aspects of proxmox.

For more [info](https://pve.proxmox.com/wiki/Command_line_tools)

# Host/Domain naming convention
A proper naming scheme should have the following pattern
```
<hostname>.<networkname>.<domain>
```

The hostname itself can having a different scheme based on the following
pattern.

For example
```
<role-location-hostid> or simple `<role-hostid>`
```
Examples of roles:

* gw - Gateway
* wl - Access point
* ap - General app server
* wb - Web server
* db - Database server
* ns - DNS
* dc - Domain
* fs - File server
* vs - Virtualization server
* gm - Game server
* nb - Notebook/Laptop
* tb - Tablet
* ph - phone

So for example based on two locations at Home and Colo I could do:
```
hmgw01.hm.example.com
clgw01.cl.example.com
```

# Proxmox Standard Locations

* Backups `/var/lib/vz/dump`
* VM installation ISOs `/var/lib/vz/template/iso`
* VM hard disk Images `/var/lib/vz/images`
* LXC container templates `/var/lib/vz/template/cache`

# An example of a good network topology
While installing pfsense, you  must first decide what kind of network topology
you want.

**Example:**

![Network topology](file:///home/adas/Documents/til/resources/matt-abr01-img03.png "Network Topology")

# Network Interface file
To manually create or edit new network interfaces on proxmox or any unix system,
the following file should be edited

```bash
$ /etc/network/interfaces file
```
# Creating a new network interface
Every interface starts with an interface definition followed by interface
options.

* **eth#** (eg: eth0, eth1, eth2) are examples of physical ethernet interfaces
* **vmbr#** (eg: vmbr0, vmbr1, vmbr2) are examples of virtual machine bridges
  * The vmbr# interfaces are like physical network switches implemented in
    software on the Proxmox VE host. All VMs can share one bridge as if virtual
    network cables from each guest were all plugged into the same switch

## A few interface options
* **auto** <interface> - starts the interface at boot time
* **inet** - interface is IPv4
* **inet6** - interface is IPv6
* **static** - static IPv4 address
* **manual** - do not define IP address for this interface. it will be used as slave for other network options
* **dhcp** - acquire IP address via dhcp protocol

For more info visit the [debian wiki](https://wiki.debian.org/NetworkConfiguration#Setting_up_an_Ethernet_Interface) about setting up an ethernet interface

## Examples
```
auto eth0
iface eth0 inet manual
```
* Start this interface at boot time (auto)
* Create/Define this interface (iface) as an interface capable of IPv4 addresses (inet)
* Do not assign any IP address to this interface (manual)

```
auto eth0
iface eth0 inet static
    address 192.0.2.7
    netmask 255.255.255.0
    gateway 192.0.2.254
```
* Start this interface at boot time (auto)
* Create/Define this interface (iface) as an interface capable of IPv4 addresses (inet)
* Assign a static(`static`) IPv4(`inet`) address configuration for this interface
* Set the IP address to 192.0.2.7 (`address`)
* Set the netmask to 255.255.255.0 (`netmask`)
* Set the gateway to 192.0.2.254 (`gateway`)

```
auto eth0
iface eth0 inet static
    address 192.0.2.7
    netmask 255.255.255.0
    gateway 192.0.2.254
    dns-nameservers 12.34.56.78 12.34.56.79
```
* Same as above except that all dns queries on this interface make use of the servers specified

```
auto vmbr0
iface vmbr0 inet static
    address 10.10.0.15
    netmask 255.255.255.0
    gateway 10.10.0.1
    bridge_ports eth0
```
* Create/Define new interface (iface) called vmbr0
* Handles IPv4 addresses (inet)
* Set the static network config (static)
* Set address, netmask and gateway since it's static
* Set the physical interface this virtual bridge is attached to (`bridge_ports`)

```
auto vmbr0
iface vmbr0 inet static
    address 10.10.0.15
    netmask 255.255.255.0
    gateway 10.10.0.1
    bridge_ports none
```
* Same as above except this virtual bridge is not attached to any physical interface (`bridge_ports none`)
* This allows you to create internal subnets that exist only within the host

# Proxmox update repos
Proxmox has 2 repos. Enterprise (paid) and Non-Enterprise (free).
If you have a subscription you can access the enterprise repos.

The enterprise subscription repo can be found at
`/etc/apt/sources.list.d/pve-enterprise.list`

To access the Non-Enterprise repo, add the following lines
```
# PVE pve-no-subscription repository provided by proxmox.com, NOT recommended for production use
deb http://download.proxmox.com/debian jessie pve-no-subscription
```
to `/etc/apt/sources.list`

If you are using Non-Enterprise repo, you can prevent error messages that occur
from accessing the enterprise repo, by commenting the following line

`# deb https://enterprise.proxmox.com/debian jessie pve-enterprise`

in the enterprise sources file.

For more [info](https://pve.proxmox.com/wiki/Package_repositories)

# Performance settings for KVMs
* CPU
  * The Sockets refers to the number of physical CPU's in the proxmox host
  * The cores refers to the number of cores per socket
  * Numa -

* Hard Disk
  * Virtio for hard disk can be used if the VM is read/write heavy
  * RAW VM image format is performant for hard disk images and good if
    read/write heavy. But the RAW VM file occupies the entire storage
    at the moment of creation.
  * If using RAW VM format, then its cache should be none.
  * qcow2 has snapshot capability and file size grows dynamically

* NIC
  * Virtio for NICs can be used if the VM is network heavy
