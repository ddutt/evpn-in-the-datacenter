# VX Simulation on Linux

This article is intended to show how to setup a simulation environment on a Linux laptop or server. More specifically, a device running Ubuntu 16.04 however these instructions can be extended to other distributions without much change.

![Reference Topology](../cldemo_topology.png)

- [Table of contents](#)
	- [Installing Virtualbox](#installing-virtualbox)
	- [Installing Git](#installing-git)
	- [Installing Vagrant](#installing-vagrant)
	- [Deploy the cldemo-vagrant VMs](#deploy-the-cldemo-vagrant-vms)
	- [Manage the cldemo-vagrant VMs](#manage-the-cldemo-vagrant-vms)
		- [Connecting to the Console with the VirtualBox GUI](#connecting-to-the-console-with-the-virtualBox-gui)
		- [Connecting to the VMs with SSH](#connecting-to-the-vms-with-ssh)
	- [Using Libvirt](#using-libvirt)

## Prerequisites

 - Ubuntu 16.04
 - Approximately 5GB disk space
 - Approximately 4GB free RAM

## Tools used

- VirtualBox [-Version 5.1.22 Installer-](http://download.virtualbox.org/virtualbox/5.1.22/virtualbox-5.1_5.1.22-115126~Ubuntu~xenial_amd64.deb)  or [(Alternate version downloads)](https://www.virtualbox.org/wiki/Downloads)
- Git 
- Vagrant [-Version 2.0.2 Installer-](https://releases.hashicorp.com/vagrant/2.0.2/vagrant_2.0.2_x86_64.deb) or [(Alternate version downloads)](https://releases.hashicorp.com/vagrant/)


## Install the tools
Install VirtualBox, Git, and Vagrant tools on the machine.  This will require admin privileges on the server.

### Installing Virtualbox

```
wget -O virtualbox_5.1.22.deb http://download.virtualbox.org/virtualbox/5.1.22/virtualbox-5.1_5.1.22-115126~Ubuntu~xenial_amd64.deb
sudo dpkg -i ./virtualbox_5.1.22.deb
```

### Installing Vagrant

```
wget -O vagrant_2.0.2.deb https://releases.hashicorp.com/vagrant/2.0.2/vagrant_2.0.2_x86_64.deb
sudo dpkg -i ./vagrant_2.0.2.deb
```

### Installing Git

```
sudo apt-get update -y && sudo apt-get install git -qy
```

## Setup the virtual topology
Time to actually do some networking, well virtual networking, OK fine it's more server stuff right now.

### Deploy the cldemo-vagrant VMs
 1. Launch a terminal shell (or SSH into your server).
 2. Clone the cldemo code locally with: `git clone https://github.com/CumulusNetworks/cldemo-vagrant.git`
 3. Change into the newly created cldemo-vagrant directory `cd cldemo-vagrant`
 4. Check the Vagrant status for the virtual machines with `vagrant status`
 5. Bring up your first VM the oob-mgmt-server with `vagrant up oob-mgmt-server`
What happens here is that Vagrant will automatically detect that you do not locally have the VM you are trying to create so it will connect to the Vagrant Cloud image store and download the image for you. This feature is one of the really powerful features of Vagrant as there are hundreds if not thousands of pre-built VMs, including Cumulus VX, available.  The oob-mgmt-server is actually built on Cumulus Vx as well.
Since this is the first time you bring up the VM the download may take a few mins to complete and then the demo sets up some tools on the server as part of the Vagrant setup.

 6. Once the Vagrant up completes, may take 5-10 mins, check the status of the VM with `vagrant status`

You should see that the 'oob-mgmt-server' VM is now in the 'running' state.

 7. Now let's bring up the oob-mgmt-switch with `vagrant up oob-mgmt-switch`
 
This step is very similar to step 5 in that Vagrant detects that the Cumulus VX image is not installed locally so it fetches the VM and installs it.

 8. Finally once the oob-mgmt-switch has completed let's bring up some more nodes in the network: `vagrant up server01 leaf01 leaf02 spine01 spine02`

### Manage the cldemo-vagrant VMs

Now that we've deployed the VMs we can get to the actual networking fun.  For this we have a couple options using the VirtualBox GUI or ssh directly into the environment.

#### Connecting to the VMs with SSH

 1. From within the cldemo-vagrant directory, run the command `vagrant ssh oob-mgmt-server`
 2. At this point you should be logged into the Out-of-Band Server as the Cumulus user. From here you can then ssh to any of other devices in the topology.
   i.e. `ssh leaf01`



## Using Libvirt
If you'd like to make use of the libvirt/kvm hypervisor that will also work.
**Make sure to use the proper Vagrantfiles by running the following command**
`cp ./Vagrantfile-kvm ./Vagrantfile` 
Specific instructions for setting up your environment for simulation using the
Libvirt/KVM hypervisor can be [found in this community article](https://getsatisfaction.cumulusnetworks.com/cumulus/topics/setting-up-an-ubuntu-16-04-server-for-simulation-with-libvirt-kvm).
