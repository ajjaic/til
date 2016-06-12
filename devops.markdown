# Network Bridges
A network bridge is like a virtual switch that is created on a host. One of
their uses is to enable VM's and containers to have access to the network and to
the wider internet. There are 2 main types of bridges,

* Host Bridge
  > The containers/VMs are directly connected to the host network and appear
    and behave as other physical machines on your network. In other words
    the host and containers/VMs are on the same network.
* NAT Bridge
  > In this case, the host and containers/VMs are on different networks. And
    network of the containers/VMs is private. So if the containers/VMs need
    access to nodes outside the host, they will have to communicate via the
    host.

# SSH into docker containers with Public Key

To `SSH` into docker containers, an ssh server must be running on the container.
And then, have a key pair on the local host. And provision the docker container
with the `public key` of the host. You can do this by ensuring that the hosts
`public key` is an authorized key in the container's `authorized_keys` file. It
can be done like by running the following command in the container,

```
$ cat host_pub_key >> ~/.ssh/authorized_keys
```
You first have to make sure that the `host_pub_key` is available on the
container.

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

Cookies are primarily a way for web servers to introduce **state** into a
stateless protocol like HTTP(S). This helps webservers to maintain state
info about a client as the client requests different resources from the sesrver.
A cookie can be set using the `Set-Cookie` HTTP header attribute.

There are a few types of cookies..

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
* If `secure` flag attribute is set, then this cookie can only be transmitted
securely (via HTTPS)

## HTTP(S)-only cookie
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
* Cookies that persist in spite of being manually deleted.
* This is done by some client side script that recreates the cookie from info
stored in multiple locations like Flash local storage, html5 storage or other
client-side storage locations.

# Aliasing An Ansible Host

When specifying the hosts that Ansible can interact with in the
`/etc/ansible/hosts` file, you can put just the IP address of the host
server, like so:

```yml
192.168.1.50
```

IP addresses are not particularly meaningful for a person to look at though.
Giving it a name serves as better documentation and makes it easier to see
what host servers are in play during a task.

Ansible makes it easy to alias host servers. For example, we can name our
host `staging` like so:

```yml
staging ansible_host=192.168.1.50
```

[source](http://docs.ansible.com/ansible/intro_inventory.html)

# Check The Status Of All Services

In a Linux environment, you can quickly check the status of a number of
different services.
```
$ sudo service --status-all
```
That will show status of all services invoked under the `/etc/init.d/`
directory. The output will contain the names of services along with differnt
symbols.

The `-` symbol means it isn't running, the `+` symbol means it is up, and the
`?` symbol means that it cannot determine the status.

# Check The Syntax Of nginx Files

The syntax of [`nginx`](https://www.nginx.com/) configuration files can be a
bit finicky. On top of that, some `nginx` server commands can fail silently.
Get more confidence by using the following command to check for syntax
errors in those files:

```bash
$ [sudo] nginx -t
```

If there is an error, you might see something like this:

```bash
$ sudo nginx -t
nginx: [emerg] unexpected ";" in /etc/nginx/nginx.conf:16
nginx: configuration file /etc/nginx/nginx.conf test failed
```

If all looks good, then you'll see this:

```bash
$ sudo nginx -t
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```

# Determine The IP Address Of A Domain

The `dig` (domain information grouper) command can be used to get more
information about a domain name. To discover the IP address for a given
domain, invoke `dig` with the domain as an argument.

```bash
$ dig joshbranchaud.com

; <<>> DiG 9.8.3-P1 <<>> joshbranchaud.com
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 26589
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 0

;; QUESTION SECTION:
;joshbranchaud.com.             IN      A

;; ANSWER SECTION:
joshbranchaud.com.      86400   IN      A       198.74.60.157

;; Query time: 118 msec
;; SERVER: 75.75.75.75#53(75.75.75.75)
;; WHEN: Sun Jan 17 22:32:18 2016
;; MSG SIZE  rcvd: 51
```

The *answer section* tells me that the IP address for `joshbranchaud.com` is
`198.74.60.157`.

See `man dig` for more details.

# Path Of The Packets

You can use `traceroute` to determine the network path packets follow from
your machine to some host server. For instance, if I want to know path
between my laptop (connected to airport wifi) and my website
[joshbranchaud.com](http://joshbranchaud.com), I can run the following
command:

```
$ traceroute joshbranchaud.com
traceroute to joshbranchaud.com (198.74.60.157), 64 hops max, 52 byte packets
 1  hotspot.boingohotspot.net (192.168.144.1)  1.209 ms  1.427 ms  1.080 ms
 2  192.168.13.1 (192.168.13.1)  5.272 ms  3.545 ms  6.231 ms
 3  10.106.144.1 (10.106.144.1)  13.671 ms  15.311 ms  12.895 ms
 4  68.13.10.184 (68.13.10.184)  14.789 ms  12.980 ms  13.628 ms
 5  68.13.8.221 (68.13.8.221)  19.922 ms  12.611 ms  17.853 ms
 6  nyrkbprj01-ae3.0.rd.ny.cox.net (68.1.5.157)  48.917 ms  56.641 ms  51.771 ms
 7  eqix.e-2-3.tbr2.ewr.nac.net (198.32.118.157)  62.977 ms  52.970 ms  48.667 ms
 8  0.e1-2.tbr2.mmu.nac.net (209.123.10.114)  66.253 ms  57.311 ms  52.242 ms
 9  207.99.53.46 (207.99.53.46)  57.478 ms  53.443 ms  54.510 ms
10  joshbranchaud.com (198.74.60.157)  59.853 ms  56.375 ms  55.393 ms
```

See `man traceroute` for more details.

# Push Non-master Branch To Heroku

When using git to deploy your app to Heroku, it is expected that you push
to the `master` branch. When you run the following command

```
$ git push heroku master
```

Heroku will attempt to build and run your app. However, if you have a
`staging` branch for your application that you want to push to your
staging environment on Heroku, you cannot simply run

```
$ git push heroku staging
```

Heroku will only perform a build on pushes to the remote `master` branch.
You can get around this, though, by specifying that your `staging` branch
should be pushed to the remote `master` branch, like so

```
$ git push heroku staging:master
```

[source](https://coderwall.com/p/1xforw/make-heroku-run-a-non-master-branch)

# Reload The nginx Configuration

If you've modified or replaced the configuration file, nginx will not
immediately start using the updated nginx configuration.
Once a `restart` or `reload` signal is received by nginx, it
will apply the changes. The `reload` signal

```
$ service nginx reload
```

tells nginx to reload the configuration. It will check the validity of the
configuration file and then spawn new worker processes for the latest
configuration. It then sends requests to shut down the old worker processes.
This means that during a *reload* nginx is always up and processing
requests.

[source](http://nginx.org/en/docs/beginners_guide.html)

# Running Out Of inode Space

Unix systems have two types of storage limitations. The first, and more
common, is a limitation on physical storage used for storing the contents of
files. The second is a limitation on `inode` space which represents file
location and other data.

Though it is uncommon, it is possible to run out of `inode` space before
running out of disk space (run `df` and `df -i` to see the levels of each).
When this happens, the system will complain that there is `No space left on
device`. Both `inode` space and disk space are needed to create a new file.

How can this happen? If lots of directories with lots of empty, small, or
duplicate files are being created, then the `inode` space can be used up
disproportionately to the amount of respective disk space. You'll need to
clean up some of those files before you can continue.

Sources: [this](http://blog.scoutapp.com/articles/2014/10/08/understanding-disk-inodes) and [this](http://www.linux.org/threads/intro-to-inodes.4130/)

# Wipe A Heroku Postgres Database

If you have some sort of non-production version of an application running on
Heroku, you may encounter a time when you need to wipe your database and
start fresh. For a rails project, it may be tempting to do `heroku run rake
db:drop db:setup`. Heroku doesn't want you to accidentally do anything
stupid, so it prevents you from running `rake db:drop`. Instead, you must
send a more explicit command

```
$ heroku pg:reset DATABASE_URL
```

Heroku will ask you to confirm that you indeed want to wipe out the database
and will then proceed.

For the curious reader, running `heroku config` will list the values of a
number of variables including `DATABASE_URL`.

