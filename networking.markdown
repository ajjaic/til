# Router vs Switch

* Routers connect networks.
* Routers route **IP packets** between different networks.
* They works at the IP layer (Level 3)

* Switches create networks
* Switches work at the MAC level (close to hardware level, level 2)
* Switches route **ethernet frames** between different devices on the same network

Whenever an ethernet frame reaches the Switch it detects if the MAC address on
the frame is destined for a MAC address that the switch knows about. If the
switch encountered that MAC address before then it will direct the frame to
the right port on the switch. But if the frame does not have a recognizable
MAC address then it will be sent to the router to be routed to the correct
network.

When a router receives an IP packet it looks at the IP address to determine the
network to which the packet needs to be routed to.
