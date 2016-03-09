# Common networking Acronyms

* `WAN` - Wide Area Network. Also known as the internet.
* `LAN` - Local Area Network. Your internal network, also known as your domain
or your intranet.
* `OPT` - Optional networks
* `DMZ` - De-Militarized Zone. A fancy name for just another type of internal
network just like your LAN. The difference is, using firewall rules you prevent
any traffic that comes into your DMZ subnet from going to your LAN subnet, for
security purposes. This is where you would host a web server or FTP server, a
place where anyone on the internet can access certain things without having
access to your private LAN devices.
* `DHCP` - A type of service that automatically hands out IP addresses. Many types
 of network devices are configurable as DHCP servers.
* `WAP` - Wireless Access Point.  Essentially, a wireless switch.
* `NIC` - Network Interface Card.

WAN, LAN, DMZ and OPT are just cannonical names for networks that perform a
certain function

# Router, Switch, Wireless Access Points
## Router
* Routers connect networks.
* Routers route **IP packets** between different networks.
* They works at the IP layer (Level 3)

## Switch
* Switches create networks
* Switches work at the MAC level (close to hardware level, level 2)
* Switches route **ethernet frames** between different devices on the same
network

## Wireless AP
* Wireless AP (aka wireless switch) is simply a wireless switch.
* The physical interface that clients have with a wireless AP the only
difference between a switch and a wireless AP

Whenever an ethernet frame reaches the Switch it detects if the MAC address on
the frame is destined for a MAC address that the switch knows about. If the
switch encountered that MAC address before then it will direct the frame to
the right port on the switch. But if the frame does not have a recognizable
MAC address then it will be sent to the router to be routed to the correct
network.

When a router receives an IP packet it looks at the IP address to determine the
network to which the packet needs to be routed to.

# Cat5, Cat5e, Cat6 Ethernet cables

## Cat5
* Cat5 is obsolete
* Older standard
* Max Speed 100 Mbps
* Bandwidth 100 Mhz

## Cat5e
* Cat5e is more modern
* `e` stands for **enhanced**
* Reduces crosstalk
* Can handle higher speeds

## Cat6
* Cat6 are more faster
* Snagless is better than non-Snagless
* Better isolation for more reduction in crosstalk

# HTTP Cookies

There are a few types of cookies

## Session cookie
* It's a in-memory cookie. Exists only as long as user navigates website.
* These cookies are deleted when browser is closed.
* Since they don't have expiration dates, the browser recognizes them as
session cookies

## Persistent Cookie
* Unlike session cookies, these have expiration dates.
* Until the cookie expires, the cookie is transmitted to the domain specified
in the domain attribute, every time user visits it.

##Secure cookie
* These cookies are only transmitted via HTTPS connections.
* If `secure` flag is set, then this cookie can only be transmitted securely

## HTTP-only cookie
* Can only be transmitted via HTTP or HTTPS.
* Ajax calls via Javascript will not transmit this cookie

## Third-party cookie
* If the domain attribute in the cookie is different compared to the domain
in the address bar, then this is a 3rd party cookie.

## Super cookie
* A super cookie has a TLD (like .com) as value for its domain attribute
* Usually they are blocked by brower
* Such cookies will be transmitted to any domain with the same TLD

## Zombie cookie
* Cookies that persist inspite of being manually deleted.
* This is done by some client side script that recreates the cookie from info
stored in multiple locations like Flash local storage, html5 storage or other
client-side storage locations.



