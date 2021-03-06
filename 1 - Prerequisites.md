# Enterprise DC/OS latest Prerequisites Installation Guide
## Updated for DC/OS 2.0, valid for DC/OS 1.13 and should be good for DC/OS 1.12 and 1.11

The prerequisites are to be run one every cluster node (Masters, Public Agent, Private Agent) including the Bootstrap node

## Compatibility Matrix

[DC/OS 1.13](https://docs.mesosphere.com/version-policy/)

## Install Nodes

[Bootstrap, Master, Private, Public Requirements](https://docs.d2iq.com/mesosphere/dcos/2.0/installing/production/system-requirements/)

NOTE: 
- Bootstrap: Is only used for cluster installation, major changes (i.e. networking), upgrading and scaling.  It can be turned off (NOT deleted) when not doing these things.
- Master: Preferred to have 3 nodes, but need an odd number (i.e. 1, 3, 5...).  If performance is critical, SSDs should be used for disk.
- Private: Larger sized.  Compute for tasks, jobs and containers.  Generally set to have no direct access to containers.
- Public: Less size.  Only should be used for direct access to containers.  LB / Ingress containers only will reside here.


## Prepare Node

### Log In and change to SU
```
sudo su
```

### Allow SUDO commands to run without password
```
sudo visudo
```
OR edit /etc/sudoers
```
%wheel ALL=(ALL) NOPASSWD: ALL
```

### OPTIONAL: Set PS1 for each node
edit /etc/bashrc (Comment out this line and add the following as below)
```
# \[ "$PS1" = "\\s-\\v\\\$ " \] && PS1="\[\u@\h \w\]\\$ "
```
Bootstrap Node:
```
PS1='BOOTSTRAP \W $ ' 
```
Master Node:
```
PS1='Master1 \W $ ' 
```
Private Node:
```
PS1='Private1 \W $ ' 
```
Public Node:
```
PS1='Public1 \W $ ' 
```

### Update Centos to 7.5 If Necessary
```
sudo cat /etc/redhat-release
sudo yum check-update
sudo yum update -y
sudo reboot
cat /etc/redhat-release
```

### Install Utility/Helper Applications
```
sudo yum install -y yum-utils xz bash net-utils bind-utils coreutils gawk gettext grep iproute util-linux curl ipset sed wget net-tools unzip
```

### Install and Configure NTP
Time synchronization is especially important for the master nodes to converge into a cluster control plane.  Please ensure that they are coirrectly synced prior to instlling the system.  Common places of trouble are excessive d=rift preventing resync, or BIOS hardware clocks that are not properly synchronized
```
sudo yum remove -y chrony
sudo yum install -y ntp
sudo systemctl enable ntpd
sudo systemctl start ntpd
sudo timedatectl set-ntp 1
```

### Install JQ
```
sudo wget http://stedolan.github.io/jq/download/linux64/jq
sudo chmod +x ./jq
sudo cp jq /usr/bin
```

### Stop # Disable "firewalld" | "dnsmasq" | SELINUX to Permissive | Add Groups | Set Locale
```
sudo systemctl stop firewalld && sudo systemctl disable firewalld
sudo systemctl stop dnsmasq && sudo systemctl disable dnsmasq.service
sudo sed -i 's/SELINUX=enforcing/SELINUX=permissive/g' /etc/selinux/config
cat /etc/selinux/config | grep SELINUX=permissive
sudo groupadd nogroup && sudo groupadd docker
sudo localectl set-locale LANG=en_US.utf8
```

### Reboot
```
sudo reboot
```

### Log Back In and change to SU
```
sudo su
```

### Install, Start, and Enable Docker
If this is your first time, perhaps [check here](https://docs.d2iq.com/mesosphere/dcos/version-policy/#centos-support-matrix-2) to make sure this documentation is up to date (10/31/2019 - yes, I work on Halloweeen :).

Docker 18.09.9 Install:
```
curl -O  https://download.docker.com/linux/centos/7/x86_64/stable/Packages/docker-ce-18.09.9-3.el7.x86_64.rpm
curl -O https://download.docker.com/linux/centos/7/x86_64/stable/Packages/docker-ce-cli-18.09.9-3.el7.x86_64.rpm
curl -O https://download.docker.com/linux/centos/7/x86_64/stable/Packages/containerd.io-1.2.6-3.3.el7.x86_64.rpm
yum -y localinstall ./containerd*.rpm ./docker*.rpm || true
systemctl start docker
systemctl enable docker

sudo docker pull nginx
sudo docker ps
```

### Reboot Again
```
sudo reboot
```
[Proceed to step "2" and prepare your bootstrap node.](https://github.com/jdyver/Enterprise-DC-OS-LATEST-Install-Cheatsheet/blob/master/2%20-%20Bootstrap_Preparation.md)

[If bootstrap is already setup proceed to step "3".](https://github.com/jdyver/Enterprise-DC-OS-LATEST-Install-Cheatsheet/blob/master/3%20-%20Installation.md)
