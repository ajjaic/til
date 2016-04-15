# Host/Domain naming convention

A proper naming scheme should have the following pattern

`<hostname>.<networkname>.<domain>`

The hostname itself can having a different scheme based on the following
pattern.

For example

`<role-location-hostid>` or simple `<role-hostid>`

Examples of roles:
* gw - Gateway
* wl - Access point
* ap - Genarl app server
* wb - Web server
* db - Database server
* ns - DNS
* dc - Domain
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
If you have a subscription you can access the enterprise repos at,

Subscription repo
```
$ cat /etc/apt/sources.list.d/pve-enterprise.list
deb https://enterprise.proxmox.com/debian jessie pve-enterprise
```

Non-Subscription repo
```
$ cat /etc/apt/sources.list
deb http://ftp.debian.org/debian jessie main contrib

# PVE pve-no-subscription repository provided by proxmox.com, NOT recommended for production use
deb http://download.proxmox.com/debian jessie pve-no-subscription

# security updates
deb http://security.debian.org/ jessie/updates main contrib
```

If you are using Non-Subscription repo, you can prevent error messages from
accessing the enterprise repo, by commenting the line in `/etc/apt/sources.list.d/pve-enterprise.list`

For more [info](https://pve.proxmox.com/wiki/Package_repositories)

# Performance settings for KVMs

* Hard Disk
  * Virtio for hard disk can be used if the VM is read/write heavy
  * RAW VM image format is performant for hard disk images and good if
    read/write heavy. But the RAW VM file occupies the entire storage
    at the moment of creation.
  * If using RAW VM format, then its cache should be none.
  * qcow2 has snapshot capability and file size grows dynamically


* NIC
  * Virtio for NICs can be used if the VM is network heavy

# Steps for creating a pfsense VM in Proxmox
