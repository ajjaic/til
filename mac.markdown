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
* **Docker Architecture in Linux**
  * ![Linux Architecture](file:///home/adas/Documents/til/resources/linux_docker_host.svg)
* **Docker Architecture in Mac OSX**
  * ![Mac OSX Architecture](file:///home/adas/Documents/til/resources/mac_docker_host.svg)
