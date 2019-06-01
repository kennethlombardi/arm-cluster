# The gist
After first burning an image to the eMMC, installing the eMMC, and then booting you'll need to simply set up the OS to have reasonable defaults.

Once the machine boots the swarming can begin

# Rock Pro 64 image burning
Use the pine installer to select the RockPro64 board and then install the emmc+sd image of Ubuntu 18.04 Bionic with DockerCE and K8s by ayufan
![asdf](docs/PINE64_RockPro64Image.png)

# Rock 64 image burning
Use the pine installer to select the Rock64 board and then install the same image type as the Rock Pro 64

# Device physical setup and login
1. Install eMMC and boot
1. Login with rock64 rock64

# Add a different user and add to sudoers
``` bash
# Add a different user:
adduser <user>

# Add that user to sudoers
sudo usermod -aG sudo <user>
```

## Perform docker post install steps
``` bash
# not needed because the image already has docker group
# sudo groupadd docker

# add user to docker group
sudo usermod -aG docker <user>

#exit session and login again to get new group setting
exit
```

## Remove old user

``` bash
# delete user and home directory:
userdel -r <user>
```

# Set up the user environment
```bash
# install and configure zsh
sudo apt install zsh
sh -c "$(curl -fsSL https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"

# clone basic environment config
git clone https://github.com/kennethlombardi/dotfiles.git
cd dotfiles

# install ruby
sudo apt install ruby

# run rakefile for basic config (this is destructive so read the readme first)
rake
# install and configure basic plugins
curl -fLo ~/.vim/autoload/plug.vim --create-dirs \
    https://raw.githubusercontent.com/junegunn/vim-plug/master/plug.vim

# vim plug install
vim
:PlugInstall

# source
source ~/.zshrc
```

# Check docker version and make sure it's at least 18.09

We need 18.09 for the `export DOCKER_HOST=ssh://HOSTNAME` command to work so we can publish remote commands with minimal setup
``` bash
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io
```

# Set the hostname to something reasonable

```bash
# get hostname and other info
hostnamectl
#  Static hostname: nanopineo2
#         Icon name: computer
#        Machine ID: 851fdc172f374ea48f95672514633def
#           Boot ID: b205ce004f384797b6b2fb75be8d7600
#  Operating System: Ubuntu 18.04.2 LTS
#            Kernel: Linux 4.19.38-sunxi64
#      Architecture: arm64

# set hostname to something reasonable
sudo hostnamectl set-hostname reasonable-name-here

# edit hosts file and replace old hostname with new
# anywhere that reads nanopineo2 should be replaced with
# reasonable-name-here
sudo vim /etc/hosts

# reboot
sudo reboot

# after logging bash in (ssh) the hostname should be set
hostnamectl
#  Static hostname: nanopineo-1
#         Icon name: computer
#        Machine ID: 851fdc172f374ea48f95672514633def
#           Boot ID: b205ce004f384797b6b2fb75be8d7600
#  Operating System: Ubuntu 18.04.2 LTS
#            Kernel: Linux 4.19.38-sunxi64
#      Architecture: arm64
```

# Generate and add ssh keys for remote commands

For every ssh login or command over ssh you will need to enter your password. This quickly gets tiring when accessing multiple hosts so we will generate a certificate and add the public key to the remote host.

Additionally when managing the swarm remotely ssh is used and the first step is `export DOCKER_HOST=ssh://HOSTNAME`. This allows every docker command on the host to be forwarded to the remote host over ssh. Without adding a certificate you will need to enter your password every time.

https://superuser.com/questions/8077/how-do-i-set-up-ssh-so-i-dont-have-to-type-my-password

https://help.github.com/en/articles/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent

https://stackoverflow.com/questions/24392657/adding-an-rsa-key-without-overwriting

# Remote commands against docker daemon
https://raesene.github.io/blog/2018/11/11/Docker-18-09-SSH/
