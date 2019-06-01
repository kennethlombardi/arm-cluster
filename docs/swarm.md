https://docs.docker.com/engine/swarm/admin_guide/

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

