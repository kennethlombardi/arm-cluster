# Nano Pi NEO2
http://wiki.friendlyarm.com/wiki/index.php/NanoPi_NEO2

# Tools
1. Image burner: https://www.balena.io/etcher/
1. IP Scanner: https://angryip.org/download/#mac
1. ⏳:

# Steps

Download `Armbian Bionic` from https://www.armbian.com/nanopi-neo-2/

On mac unzip the .7z and observe a .img is present in the unzipped folder

Burn the image using Balena Etcher to a micro sd card

Insert the sd card into the nano pi

Plug a network cable into the nano pi so it can get an ip address upon boot using DHCP. You can set a static IP later.

Power the nano pi using the micro usb cable. Make sure the power supplied is at least 2A.

Find the ip address of the nano pi using your favorite method such as logging into your DHCP server or router and observing if an ip address was provided for a device named `nanopineo`.

Alternatively find the IP address using the angry ip scanner. See the tools section.

SSH into the nano pi. Default username is `root` and the password is `1234`.
> `ssh root@remote-ip-you-found-earlier`

You will be asked for the current password upon login and then immediately prompted to change the default root password as well as to create a new user. The new user will be added to the `sudo` group.

Exit the session and login with your new user.

# Change hostname to something reasonable

The hostname should default to something like `nanopineo2`. Since it is very likely we're going to have more than one of these we need to rename them to something that scales.

See the section in the root [README](../README.md) concerning changing the hostname. Currently titled `Set the hostname to something reasonable` and replace `<reasonable-name-here>` with `nanopineo-1` or whatever number you want.

# Set a static IP

Option 1 is to use your router DHCP server. Option 2 is to set a static IP on the device itself. Option 1 is simpler and easier to understand for beginners. Option 2 is more difficult but is a standard practice for most administrators but can cause an unreachable host in the case of IP address conflicts in misconfigured or mismanaged networks such as when two devices ask for the same IP address.

Choose the option that you're most comfortable with. In the end we just need a consistent and easy way to keep track of our node on the network that won't change between boots.

## Option 1: Router DHCP

Log in to your router and set a reserved IP for the new nano pi based on mac address in the DHCP settings. Every router is different so look up specifics for your manufacturer.

## Option 2: Set a static IP address in `/etc/network/interfaces`

https://linuxconfig.org/how-to-setup-a-static-ip-address-on-debian-linux

# Install docker

In order to add this node to a swarm it will need to have docker installed.

Test the version of docker using `docker info`. The response should be
```
Command 'docker' not found, but can be installed with:

sudo apt install docker.io
```

```bash
# install docker
sudo apt install docker.io -y

# add user to docker group so sudo isn't necessary
sudo usermod -aG docker your-user

# log out and log back in to get group changes
exit

# make sure docker responds properly
docker --version # Docker version 18.09.2, build 6247962
docker run --rm hello-world
```

 Note on adding user to the docker group
> Adding a user to the “docker” group grants them the ability to run containers which can be used to obtain root privileges on the Docker host. Refer to [Docker Daemon Attack Surface](https://docs.docker.com/engine/security/security/#docker-daemon-attack-surface) for more information.

# Commands

```bash
# Get current version of OS
lsb_release -a

#
```

