# Ambari on Docker - multi host using Bare-Metal Docker Swarm Clustering

This projects aim is to help you to get started with multi-host Ambari on Bare-Metal Docker Swarm Clustering. Using a hetergenous set of physical machines you can create a Docker Cluster that can continue to expand over time.  Instructions will be given for Ubuntu 14.2 right now, others to follow. 

## Bare-Metal Docker Swarm Clustering for Ambari Hadoop Cluster

Bare-Metal Docker Swarm Clusters are built using multiple physical hosts that have Docker and Swarm installed locally, not through Docker Machine.  
## What is needed - Infrastructure
* More than one host with linux (Ubuntu 14.x) installed.  They can be virutal machines, but if you are going to use VM's you may want to use Docker Machine which is well explained in the Docker documentation.
* All hosts must have either a fixed IP or an IP that will not change
* All hosts must not have a DNS installed
## Why do I use Consul
* Consul is used for two reasons:
  * Consul is used as a dynamic DNS for the Ambari nodes
  * Consul is used as a "backing store" for the Docker Cluster
## What should be installed onto each host - Software
  * [Go Language](https://golang.org/)
  * [Go Dep](https://github.com/tools/godep)
  * [Docker](http://www.docker.com)
  * [Docker Swarm](https://github.com/docker/swarm)
  * [Consul](https://www.consul.io/intro/getting-started/install.html)
## Optional Software
  * [Docker Compose](https://github.com/docker/compose)
# Prepare the hosts
## Add each host to the /etc/hosts file with a short name for the machine
## Pick which host will start the Consul 
## Create an upstart on each machine so that when it comes up
* There is documentation at ([Docker](http://www.docker.com) for implementing a Docker Swarm cluster
