# Network isolation modules for [Apache Mesos](http://mesos.apache.org)

The first implementation in this repository showcases Apache Mesos using Project Calico as the networking solution. 

We recommend running the demo from a Linux host, specifically Ubuntu 14.04-desktop. A Vagrant file has been provided to create this environment.

## Setup
### Vagrant Setup

1. Download and install VirtualBox and Vagrant.

2. Clone this repository.

        git clone https://github.com/mesosphere/net-modules.git

3. Start the Vagrant VM.  This will automatically provision the VM.  Sit back and relax, it takes a few minutes to pre-load the Docker images used for the demo.

        cd net-modules
        vagrant up

4. Ensure you wait until the Vagrant script has completed succesfully before [moving onto the Demo](#demo).

_Note: the shell provision step contains steps that must be performed each time the VM is booted.  Append the `--provision-with shell` flag when running `vagrant up` or `vagrant reload` when booting the VM subsequent times._

### Linux Setup

1. Install Docker: https://docs.docker.com/installation/

2. Install Docker-compose:  https://docs.docker.com/compose/install/

3. Load Kernel modules used by Project Calico:

        sudo modprobe ip6_tables
        sudo modprobe xt_set

4. Clone this repository.

        git clone https://github.com/mesosphere/net-modules.git

## Demo
Vagrant users should run all demo functionality from within their Ubuntu VM.

### Build the demo (Vagrant and Linux)
The demo runs several docker images to simulate a full Mesos Cluster:

| Image       | Description          |
|-------------|----------------------|
| mesosmaster | mesos master         |
| slave       | mesos slave          |
| marathon    | mesos app framework  |
| zookeeper   | datastore for Mesos  |
| etcd        | datastore for Calico |

To download and build these docker images, enter the `net-modules` directory and run:

    make images

### Run the "before" demo
The user story is that we have an application called Probe which must run on port 9000.
The user requirement is to run 4 instances of Probe application on our cluster.
Our cluster has two servers to run Probe.
Mesos is setup with two agents - one agent per server.

#### Setup the Cluster
This is the vanilla Mesos experience : **port conflicts and no network isolation.**
- Launch the cluster
```
./demo/launch-cluster-before.sh
```
- Check Marathon at  `http://localhost:8080/ `   
Marathon should show that there are no available applications - this is because we have not launched any applications at this point, just created the hardware and agents.
- Check Messos   
The Mesos status page is available at `http://localhost:5050/ `     
The home Mesos tab should show no Active Tasks and no Completed Tasks.
The Mesos Slaves tab should show two slaves available.

#### Launch The Tasks
There are five tasks.  There are four copies of the Probe application called probe-a, probe-b, probe-c, probe-d.  There is a visualization application called collect
- Create the Marathon Tasks
```
./demo/launch-probes-before.sh
```
- Check Marathon  at  `http://localhost:8080/ `   
Marthon should now list the 5 applications.  It should show that the collect application is healthy, two probe applications are healthy and two probe applications are not healthy
- Check Messos   
The Mesos status page is available at `http://localhost:5050/ `.   
The home Mesos tab should show three Active Tasks.  These are the visualization app and two probe applications.

#### Find the IP Address of the Visualizer Application
- The IP address for Visualizer can be read directly from Mesos - it is the host value for the collect task.
- We can get the same value from docker using the following command  `docker inspect --format '{{ .NetworkSettings.IPAddress }}' netmodules_slave_1`

#### Check Connectivity between the Probe Application
To check the connectivity between the running application use the IP address from the previous step and go to port 9001.  The address should be like  `http://<Visualizer_app_IP>:9001/`.  

#### Tear down the cluster for your next demo.
```
./demo/stop-cluster-before.sh
```

#### Conclusions
You should see only two probes are running, since multiple probes cannot bind to the same port on the same host.
### Run the Calico demo w/o isolation demo

This demo shows Calico without network isolation.  All probes are assigned their own IP Address and can reach one another.

1. Launch the cluster

        ./demo/launch-cluster.sh

2. Wait until the cluster is up, then launch the probes

        ./demo/launch-probes.sh

3. View the Stars Visualizer by visiting http://192.168.255.253:9001/.
    > Since each probe has its own IP, we can view the Stars Visualizer by directly navigating to the IP it was statically assigned.


### Run the Calico demo w/ isolation demo
1. Using the same cluster, launch the test probes and targets with isolation.

        ./demo/launch-probes-isolated.sh

2. Verify by visiting the Isolated Stars Visualizer page: http://192.168.255.252:9001/

3. Tear down the test workloads.

        ./demo/stop-probes.sh
        ./demo/stop-probes-isolated.sh

   Or, alternatively simply tear down the cluster.

        ./demo/stop-cluster.sh
