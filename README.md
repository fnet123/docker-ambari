# Ambari on Docker - using multiple physical hosts for Bare-Metal Docker Swarm Clustering

This project enables you to get started with multi-host Bare-Metal Docker Swarm Clustering. It employs this approach to deploy Ambari and Hadoop clusters. Using a heterogenous set of physical machines you can create a Docker Cluster that can continue to expand over time.  

The instructions for Ambari can be extended to any other service that provided clustering such as Apache Tomcat, Apache Nifi, Apache Spark, and Apache Storm.

## Bare-Metal Docker Swarm Clustering for Ambari Hadoop Cluster

Bare-Metal Docker Swarm Clusters are built using multiple physical hosts that have Docker and Swarm installed locally, not through Docker Machine.  
## What is needed to make this work?
* More than one host with linux (Ubuntu 14.x) installed.  They can be virutal machines, but if you are going to use VM's you may want to use Docker Machine which is well explained in the Docker documentation.
* All hosts must have either a fixed IP or an IP that will not change
* All hosts must not have a DNS installed

## Why do I use Consul for DNS?
* Consul is used for three reasons:
  * __It is easy to install and use with Docker__ - Consul's documentation on setting it up for use is great and it has a very nice GUI.  In addition it has greater ability to handle failover that may be nice to use in the future.
  * __Consul is used as a dynamic DNS for the Ambari nodes__ - Hadoop and many other network based software needs to have a fully qualified domain name (FQDN) to operate. Consul provides this.  The important detail is that you have to create a Consul server with Port 53 exposed, which is used for DNS in order to have the Docker nodes use it.  If you have dynamic DNS already installed in your environment you can use it as well, but you will need to find a way to add the containers dynamically at run-time.
  * __Consul is used as a "backing store" for the Docker Cluster__ - Docker requires that if you want to use Docker Swarm you must have a "backing store."  This is explained in great detail on the [Docker Site](https://www.docker.com)

## Pre-requisites for each host
  * [Go Language](https://golang.org/) - Make sure to install the latest version of Go onto your hosts.  If you do not go Dep and Swarm will not compile. 
  * [Go Dep](https://github.com/tools/godep) - Follow the instructions to download and install.  You will need to set a couple of environment variables (GOROOT and GOCODE) before running the instructions.
  * [Docker](http://www.docker.com) - Follow the instructions for your Linux variant, you should make sure to use atleast the Docker 1.9+ version.  When installing Docker you will have a startup script installed so that you can access it as a service.  You will need to follow the instructions on the Docker Swarm install to modify the startup for Docker Swarm.
  * [Docker Swarm](https://github.com/docker/swarm) - Follow the instructions to download and install Swarm.  Once you have compiled Swarm copy the executable under ${GOCODE)/bin to /usr/bin (or the appropriate standard directory).
  * [Consul](https://www.consul.io/intro/getting-started/install.html) - Follow the instructions to install Consul.  If you are not deploying this to a Production environment you may not need more than one host running Consul.  Make sure that you expose port 53 for DNS (instructions are on the Consul site).
  
## Optional Installs
  * [Docker Compose](https://github.com/docker/compose) - Same as Docker Swarm, you will want to copy the executable into /usr/bin.  Docker Compose is not used in the scripts but is useful.

## Prepare the hosts
 
* Add each host to the /etc/hosts file with a short name for the machine
* Pick which host(s) will start the Consul service
* Create a startup script (/etc/init/consul.conf) on the consul server machine so that when it comes up it will start up consul.
* Decide which host(s) will have a Docker Swarm Manager.  You will want to use a port that is easy to remember as you will use it on each Docker call once the cluster is usable. 
* Modify the Docker startup script on each host to advertise itself as part of the Docker Swarm Cluster.
* Create a startup script to start Swarm on each host.
* Create a startup script to Start the Swarm Manager on the host(s).
* When you have finished setting up the Docker Swarm, setup the [Docker Overlay Network](https://docs.docker.com/engine/userguide/networking/dockernetworks/).

## Running Ambari and Hadoop using Docker Swarm
* Export the location of your Docker Swarm Manager:

`$export DOCKER_MANAGER=tcp://192.168.1.18:2376`

* Export the name of the Docker Overlay Network you are using

`$export OVERLAY_NETWORK=my-net`

* Export the location of the Consul Server

`$export CONSUL=192.168.1.18`

* Put the ambari functions into your environment

`$ . ambari-functions`

* Start the ambari server and nodes.  The command below will start an Ambari cluster with 5 nodes, one for the Ambari server and 4 nodes with Ambari agents.

`$ amb-start-cluster 5`

* Find the host where the Ambari server container has been created:

`$ get-swarm-host amb-server`

`$ ubuntu-hp-jdavis`

* Use the Ambari User Interface to create your cluster by going to the server to the server you received from the above query.  If you have setup all of your hosts in /etc/host on each machine you will be able to access the server in your browser.  Right now the port exposed for the Ambari server is 8182, you may need to change that if this port is already used on all of your hosts.  Docker will not put this container onto a host where the port is already used.
  * In my environment the ambari server would be accessed using:

  `http://ubuntu-hp-jdavis:8182/#/login`
  
* At this point you can install your Hadoop server onto your Ambari cluster! 

## Adding a Host
### Install your favorite Linux variant onto the host
[Ubuntu 14.04 trusty](http://releases.ubuntu.com/14.04/) is the Linux OS that has been used to build this, but you can [pick](https://docs.docker.com/engine/installation/) any Linux OS that is compatible with the latest version of Docker.  The one requirement is that it must be a 64 bit Operating System.

### Install these standard Linux packages
`$sudo apt get install git`

### Install Docker Daemon
* Follow instructions here for [Ubuntu Linux](https://docs.docker.com/engine/installation/ubuntulinux/), it is very important that you have a version of Docker installed that is equal or greater than 1.9.x.
* As part of the installation the installation will create a startup script, in Ubuntu this will be in`/etc/init/Docker`.
* In later steps we will be changing this file so you will need to know where this file is.

### Install Consul
* You can install Consul on all of your hosts, or just one.  So, if it's already installed on one of your hosts, this is an optional installation.
* My recommendation is to use the instructions [here](https://www.digitalocean.com/community/tutorials/how-to-configure-consul-in-a-production-environment-on-ubuntu-14-04).
* Install Consul as a service, here is an example configuration file:
 
`  {                                 `

`    "bootstrap": true,  `

`    "server": true, `

`    "data_dir": "/var/consul",`

`    "encrypt": "<Put your encrypted value here>",`

`    "log_level": "INFO",`

`    "enable_syslog": true,`

`    "ui_dir": "/ui",`

`    "client_addr": "0.0.0.0",`

`    "ports": {`

`             "dns": 53`

`            },`

`    "recursor": "8.8.8.8",`

`    "disable_update_check": true`

`}`


### Install Go
* Execute these commands: `$wget https://storage.googleapis.com/golang/go1.5.3.linux-amd64.tar.gz` `$sudo tar -C /usr/local -xzf go1.5.3.linux-amd6.tar.gz`
* Make a directory for the code in the home directory of the user that will comple Go code.
`mkdir gocode`
* Edit the `.profile` or `.bash_profile` depending on the Linux variant and add this variable to the user that will compile the Swarm code.  
`export PATH=$PATH:/usr/local/go/bin`

 `GOROOT=/usr/local/go`
 
 `GOPATH=$HOME/gocode`
 
### Install Godep
* Follow the instructions [here](https://github.com/tools/godep).

### Install Swarm
* Follow the instructions [here](https://github.com/docker/swarm) to compile Docker Swarm.  If you have an earlier version of Go installed lower than 1.9 you will have compile errors in that situation you should follow the instructions, uninstall your current version and add the new version.
* Copy the swarm executable from the `${GOCODE}/bin` directory to `/usr/bin` `sudo cp ${GOCODE}/bin/swarm /usr/bin/.`

 `sudo chmod 755 /usr/bin/swarm`

* Create a startup script in `/etc/init` called swarm.conf a sample is below:

`description "Docker Swarm daemon"`

`start on (local-filesystems and net-device-up IFACE!=lo)`

`stop on runlevel [!2345]`

`limit nofile 524288 1048576`

`limit nproc 524288 1048576`

`respawn`

`kill timeout 20`

`pre-start script`

`end script`

`script`

`        # modify these in /etc/default/$UPSTART_JOB (/etc/default/docker)`

`        SWARM=/usr/bin/swarm`

`        SWARM_OPTS="--advertise=<hostip>:2375 consul://<consulIp>:8500"`

`        if [ -f /etc/default/$UPSTART_JOB ]; then`

`                . /etc/default/$UPSTART_JOB`

`        fi`

`        exec "$SWARM" join $SWARM_OPTS`

`end script`

`post-start script`

`        DOCKER_OPTS=`

`       echo "swarm join is up"`

`end script`