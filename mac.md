# Running docker on Mac OSX
Architecturally docker consists of a few componets

* Docker host
  * The host is the machine where the docker daemon is running
* Docker daemon
  * The daemon is the one that manages all the docker containers and images
* Docker client
  * The client is used for interacting with the daemon
  * The daemon and client do not have to run on the same machine
  * The client can communicate via the network with the daemon
* Docker Architecture in Linux
  * ![Linux Architecture](file:///home/adas/Documents/til/resources/linux_docker_host.svg)
* Docker Architecture in Mac OSX
  * ![Mac OSX Architecture](file:///home/adas/Documents/til/resources/mac_docker_host.svg)

# Default Screenshot Location
By default, Mac saves all screenshots to the desktop. If you'd like
screenshots to be dumped somewhere else, you have to configure it manually
from the command line. For instance, if you'd like your screenshots to be
saved in the `~/screenshots` directory, then enter the following commands:

```bash
$ mkdir ~/screenshots
$ defaults write com.apple.screencapture location ~/screenshots
$ killall SystemUIServer
```
[source](http://osxdaily.com/2011/01/26/change-the-screenshot-save-file-location-in-mac-os-x/)

# Switch Versions of a Brew Formula
If you've installed a couple versions of a program via brew and you'd like
to switch from the currently linked version to the other installed version,
you can use the `switch` command. For instance, if you are on version
`1.8.2` of `phantomjs` and you'd like to switch to `1.9.0`, you can simply
invoke:
```
$ brew switch phantomjs 1.9.0
```

More generically:
```
$ brew switch <formula> <version>
```
