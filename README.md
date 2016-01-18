# Ambari on Docker - multi host using Bare-Metal Docker Swarm Clustering

This projects aim is to help you to get started with multi-host Ambari on Bare-Metal Docker Swarm Clustering. Using a hetergenous set of physical machines you can create a Docker Cluster that can continue to expand over time.  Instructions will be given for Ubuntu 14.2 right now, others to follow. 

## Bare-Metal Docker Swarm Clustering for Ambari Hadoop Cluster

Bare-Metal Docker Swarm Clusters are built using multiple physical hosts that have Docker and Swarm installed locally, not through Docker Machine.  
### What is needed - Infrastructure
* More than one host with linux (Ubuntu 14.x) installed.  They can be virutal machines, but if you are going to use VM's you may want to use Docker Machine which is well explained in the Docker documentation.
* All hosts must have either a fixed IP or an IP that will not change
* All hosts must not have a DNS installed
### Why do I use Consul
* Consul is used for two reasons:
** Consul is used as a dynamic DNS for the Ambari nodes
** Consul is used as a "backing store" for the Docker Cluster
### What should be installed onto each host - Software
* [Go Language](https://golang.org/)
* [Go Dep](https://github.com/tools/godep)
* [Docker](http://www.docker.com)
* [Docker Swarm](https://github.com/docker/swarm)
* [Consul](https://www.consul.io/intro/getting-started/install.html)

### What are the steps for sucessful host install?
# Prepare the hosts
## Add each host to the /etc/hosts file with a short name for the machine
## Decide which host will start the Consul 
## Create an upstart on each machine so that when it comes up
* There is documentation at ([Docker](http://www.docker.com) for implementing a Docker Swarm cluster


## Starting the container

This will start (and download if you never used it before) an image based on
Centos 7 with pre-installed Ambari 2.2.0 ready to install HDP 2.3.

This git repository also contains an ambari-functions script
which will launch all the necessary containers to create a fully functional cluster. Download the file and source it:
```
. ambari-functions or source ambari-functions
```
Now you can issue commands with `amb-`prefix like:
```
amb-settings
```
To start a 3 node cluster:
```
amb-start-cluster 3
```
It will launch containers like this (1 Ambari server 2 agents 1 consul server):
```
CONTAINER ID        IMAGE                          COMMAND                  STATUS              NAMES
52b563756d26        hortonworks/ambari-agent       "/usr/sbin/init syste"   Up 9 seconds        amb2
ddfc8f00d30a        hortonworks/ambari-agent       "/usr/sbin/init syste"   Up 10 seconds       amb1
ca87a0fb6306        hortonworks/ambari-server      "/usr/sbin/init syste"   Up 12 seconds       amb-server
7d18cc35a6b0        sequenceiq/consul:v0.5.0-v6   "/bin/start -server -"    Up 17 seconds       amb-consul
```

Now you can reach the Ambari UI on the amb-server container's 8080 port. Type the `amb-settings` for IP:
```
amb-settings
...
AMBARI_SERVER_IP=172.17.0.17
```

## Cluster deployment via blueprint

Once the container is running, you can deploy a cluster. Instead of going to
the webui, we can use ambari-shell, which can interact with ambari via cli,
or perform automated provisioning. We will use the automated way, and of
course there is a docker image, with prepared ambari-shell in it:

```
amb-shell
```

Ambari-shell uses Ambari's [Blueprints](https://cwiki.apache.org/confluence/display/AMBARI/Blueprints)
capability. It posts a cluster definition JSON to the ambari REST api,
and 1 more json for cluster creation, where you specify which hosts go
to which hostgroup.

Ambari shell will show the progress in the upper right corner.

## Multi-node Hadoop cluster

For the multi node Hadoop cluster instructions please take a look at [Cloudbreak](http://hortonworks.com/hadoop/cloudbreak/).

If you don't want to check out the project from github, then just dowload the ambari-fuctions script, source it and deploy a
an Ambari cluster:
```
curl -Lo .amb j.mp/docker-ambari && source .amb && amb-deploy-cluster
```

