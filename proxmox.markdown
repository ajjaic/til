
While installing pfsense, you  must decide what kind of network you want.

* DMZ (WAN ACCESS SITES)
* LAN ACCESS SITES
* PROXMOX MANAGEMENT

Network configuration

* The following needs to be done in the /etc/network/interfaces file
* Create the physical interfaces (eth#)
* Create the virtual machine bridges (vmbr#). The vmbr# interfaces are like physical network switches implemented in software on the Proxmox VE host. All VMs can share one bridge as if virtual network cables from each guest were all plugged into the same switch

Interface options
* auto <interface> - starts the interface at boot time
* inet - interface is IPv4
* inet6 - interface is IPv6
* static - static IPv4 address
* manual - do not define IP address for this interface. it will be used as slave for other network options
* dhcp - acquire IP address via dhcp protocol

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



For more [info](https://wiki.debian.org/NetworkConfiguration#Setting_up_an_Ethernet_Interface)
