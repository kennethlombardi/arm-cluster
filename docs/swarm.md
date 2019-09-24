# Official docs

Tutorial: https://docs.docker.com/engine/swarm/swarm-tutorial/
Admin guide: https://docs.docker.com/engine/swarm/admin_guide/

The steps below differ from the docker swarm tutorial by
1. Not using docker machine
1. Simplifying the steps to a minimum
1. Adding notes based on my experience

# Getting started

The steps will be
1. Initialize a swarm
1. Join the swarm as a manager
1. Join other managers using manager join token
1. Join workers to swarm
1. Schedule some work by deploying a stack

## Initialize the swarm and join as manager

First log into any node you'll want as a manager and run `docker swarm init`

If you get a warning like
``` bash
Error response from daemon: could not choose an IP address to advertise since this system has multiple addresses on interface eth0 (numbers and numbers) - specify one with --advertise-addr
```
Then do what it says a-la
`docker swarm init --advertise-addr ipaddress:2377` where the ip address is the public facing ip address of your node

Once you successfully run either of the join commands the current machine should be added as a manager and node. The output will hand you output similar to
```bash
To add a worker to this swarm, run the following command:

    docker swarm join --token SWMTKN-1-numbers ipaddress:2377

To add a manager to this swarm, run 'docker swarm join-token manager' and follow the instructions.
```

Confirm the swarm is active and the current machine is set as a manager by performing a `docker info` and looking for

```bash
Swarm: active
 NodeID: nodeid
 Is Manager: true
 ClusterID: clusterid
 Managers: 1
 Nodes: 1
```
## Join other managers with join token

In the previous step the output from the swarm initialization gave us two commands to run.
```bash
# add a worker
docker swarm join --token SWMTKN-1-numbers ipaddress:2377

# get a manager join command
docker swarm join-token manager
```

We want to add more managers so run `docker swarm join-token manager`. The output should be close to

```bash
To add a manager to this swarm, run the following command:

    docker swarm join --token SWMTKN-1-numbers ipaddress:2377
```

SSH into your other manager you want to join into the managers and run the command provided from `docker swarm join-token manager`

```bash
machine1> docker swarm join-token manager
To add a manager to this swarm, run the following command:

    docker swarm join --token SWMTKN-1-numbers ipaddress:2377

machine1> ssh machine2
machine2> docker swarm join --token SWMTKN-1-numbers ipaddress:2377
This node joined a swarm as a manager.
```

Seeing `This node joined a swarm as a manager.` is our success case.

Rinse and repeat for all nodes you want as managers. Keep in mind docker swarm enjoys odd numbers of managers to maintain a raft based quorum.

See https://docs.docker.com/engine/swarm/join-nodes/ and https://docs.docker.com/engine/swarm/admin_guide/ for a note on Raft and manager numbers.

> Manager numbers note: A quorum majority is `(#nodes / 2) + 1`.
> Manager numbers note: Fault tolerance is  `(#nodes - 1) / 2`.

> 1 node: `(1/2) + 1 = 1` meaning 1 node needs one node to have a majority. Fault tolerance is 0 meaning you lose one node and your cluster is down (majority lost when `1 - 1 = 0` and required consensus majority requires `1`)

> 2 nodes: `(2/2) + 1 = 2` meaning 2 nodes needs 2 nodes to have a majority. Fault tolerance is 0 meaning you lose one node and your cluster is down (majority lost when `2 - 1 = 1` and required consensus majority requires `2`)

> 3 nodes: `(3/2) + 1 = 2` meaning 3 nodes needs 2 nodes to have a majority. Fault tolerance is 1 meaning you can lose 1 node. (majority maintained with 2 remaining active nodes).

> 4 nodes: `(4/2) + 1 = 3` meaning 4 nodes needs 3 nodes to have a majority. Fault tolerance is 1 meaning you can lose 1 node. (majority maintained with 3 remaining active nodes after losing 1)

> 5 nodes: `(5/2) + 1 = 3` meaning 5 nodes needs 3 nodes to have a majority. Fault tolerance is 2 meaning you can lose 2 nodes. (majority maintained with 3 remaining nodes after losing 2)

As you can see extra fault tolerance comes only at the odd numbers: 1, 3, 5, 7 etc. Adding an even number of nodes does not increase fault tolerance. Having an even number may be convenient for adding the last odd as a manager and then retiring a different manager all while maintaining expected fault tolerance levels.

## Join workers to swarm

Now that we have at least one manager let's deal with workers.

The steps
1. Mark managers as unwilling to work
1. Add a worker

### 1.  Mark manager as drain aka `unwilling to do work`

When adding a manager it is also a worker by default so really you already have a manager and a worker. However, it is bad practice to schedule work on a manager unless you can guarantee the work scheduled has resource limits. In most cases it is advisable to not have managers also be workers.

See https://docs.docker.com/engine/swarm/swarm-tutorial/drain-node/

Running `docker node list` should return a list of all the current nodes and their status as a worker or manager
```bash
machine1> docker node list
ID                            HOSTNAME            STATUS              AVAILABILITY        MANAGER STATUS      ENGINE VERSION
7xqsc1yqdvsjs1wy6dk2ub6ds     nanopineo2          Ready               Active              Reachable           18.09.7
k8frcm12sf5ywkgj6645j2822 *   nanopineo2          Ready               Active              Leader              18.09.7

```

The status listed above shows both hosts as managers and ready to actively accept work.

Remove the manager's willingness to accept work by issuing a draining availability command for each manager. Logging into each manager isn't necessary as every manager can manage the swarm workers and managers.

```bash
# docker node update --availability drain <node-hostname or id-here>
machine1> docker node update --availability drain 7xqsc1yqdvsjs1wy6dk2ub6ds
7xqsc1yqdvsjs1wy6dk2ub6ds

machine1> docker node update --availability drain k8frcm12sf5ywkgj6645j2822
k8frcm12sf5ywkgj6645j2822
```

Doing another `docker node list` should return the status of your managers as ready with a Drain availability meaning they aren't willing to accept work.

```bash
machine1> docker node list
ID                            HOSTNAME            STATUS              AVAILABILITY        MANAGER STATUS      ENGINE VERSION
7xqsc1yqdvsjs1wy6dk2ub6ds     nanopineo2          Ready               Drain               Reachable           18.09.7
k8frcm12sf5ywkgj6645j2822 *   nanopineo2          Ready               Drain               Leader              18.09.7
```

### 2. Add a worker

Now that we have at least one manager that is also set with an availability of Drain there's nobody left to do the actual work. The last step will be to add a worker to the swarm.

Option 1: Scrolling back through your command line history will show the command that looked like the below.
```bash
To add a worker to this swarm, run the following command:

    docker swarm join --token SWMTKN-1-numbers ipaddress:2377
```

Option 2: Issue a command to get the worker join command and token again
```bash
machine1> docker swarm join-token worker
docker swarm join --token SWMTKN-1-numbers ipaddress:2377
```

> Any manager can issue the join token

SSH into a machine that isn't a manager and paste in the `docker swarm join...` command

```bash
workermachine> docker swarm join --token SWMTKN-1-numbers ipaddress:2377

This node joined a swarm as a worker.
```

> If you get a warning about the node already being part of the cluster follow the instructions and do a `docker swarm leave` and reissue the join command

On any manager node issue `docker node list`
```bash
machine1> docker node list
ID                            HOSTNAME            STATUS              AVAILABILITY        MANAGER STATUS      ENGINE VERSION
g8gke3emx9rs0xm63mwd4r1et     leela               Ready               Active                                  18.09.6
7xqsc1yqdvsjs1wy6dk2ub6ds     nanopineo2          Ready               Drain               Reachable           18.09.7
k8frcm12sf5ywkgj6645j2822 *   nanopineo2          Ready               Drain               Leader              18.09.7
```

You will see your manager(s) listed as Drain and your worker listed as Active. Managers also have a manager status with at least one Leader

> If the docker versions are too different on each node you may encounter issues. Attempt to get the docker versions as close as possible to avoid compatibility problems.

## Schedule some work by deploying a stack

The whole point of setting up our cluster is to schedule work on them. The simplest way to schedule work would be to spin up a single container on one of the nodes and be done with it. However we want to schedule work generically using words like `service` and `stack`.

See https://docs.docker.com/engine/swarm/swarm-tutorial/deploy-service/ and https://docs.docker.com/engine/swarm/stack-deploy/ for service and stack information.

At this point you can follow the rest of the swarm tutorial at
https://docs.docker.com/engine/swarm/swarm-tutorial/deploy-service/

The key command to show success being: `docker service create --replicas 1 --name helloworld alpine ping docker.com`

And then: `docker service ls`

# Commands
```bash
docker service create --replicas 1 --name helloworld alpine ping docker.com

docker service ls

docker service inspect --pretty <SERVICE-ID>

docker service inspect helloworld

docker service ps helloworld

# Only on node running the container
docker ps

docker service scale helloworld=5

docker service rm helloworld
```

## Debugging
telnet {host} {port}

docker swarm init command with the --force-new-cluster
docker swarm init --force-new-cluster

--force or -f flag with the docker service update
docker service update --force

journalctl -u docker.service

docker node update --availability drain <node-name-here>

docker node inspect --pretty worker1

docker node inspect manager1 --format "{{ .ManagerStatus.Reachability }}"

# Join as a drain manager
docker swarm join --availability=drain --token SWMTKN-1-2345 <ip>:2377


# enable experimental features

## enable experimental daemon features
Edit `/etc/docker/daemon.json`
sudo vim /etc/docker/daemon.json
```
{
    "experimental": true
}
```

## enable experimental cli features
Edit  /etc/docker/daemon.json
``` bash
mkdir ~/.docker
sudo vim ~/.docker/config.json
```

Paste in
``` text
{
        "experimental": "enabled"
}
```
https://github.com/docker/cli/issues/947

https://github.com/docker/cli/issues/947#issuecomment-373306872

## restart the daemon
sudo systemctl restart docker

