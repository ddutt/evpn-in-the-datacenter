# VX Simulation on Mac OS X

This article is intended to show how to setup a simulation environment on a Mac OS X computer.

![Reference Topology](../cldemo_topology.png)

- [Table of contents](#)
    - [Installing XCode & XCode Tools](#installing-xcode-and-xcode-tools)
    - [Installing Homebrew](#installing-homebrew)
	- [Installing VirtualBox](#installing-virtualbox)
	- [Installing VirtualBox Extension Pack](#installing-virtualbox-extension-pack)
	- [Installing Vagrant](#installing-vagrant)
	- [Deploy the cldemo-vagrant VMs](#deploy-the-cldemo-vagrant-vms)
	- [Manage the cldemo-vagrant VMs](#manage-the-cldemo-vagrant-vms)
		- [Connecting to the Console with the VirtualBox GUI](#connecting-to-the-console-with-the-virtualBox-gui)
		- [Connecting to the VMs with SSH](#connecting-to-the-vms-with-ssh)

## Prerequisites

 - Mac OS X 10.11
 - Approximately 5GB disk space
 - Approximately 4GB free RAM

## Tools used

- XCode & XCode Tools
- Homebrew
- VirtualBox & VirtualBox Extension Pack
- Git
- Vagrant

## Installing the tools
Install XCode, XCode Tools, Homebrew, VirtualBox, VirtualBox Extension Pack and Vagrant.

### Installing XCode and XCode Tools

Click link to Get Xcode —>

https://itunes.apple.com/au/app/xcode/id497799835?mt=12

Agree to EULA and Install.

Install Prerequisite Software (XCode Tools from Command Line): Open a Terminal (Launchpad —> Other —> Terminal). At the command prompt run the following command to install XCode Tools

```
xcode-select —install
```

### Installing Homebrew

```
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

### Installing VirtualBox

```
brew cask install virtualbox
```

### Installing VirtualBox extension pack

```
brew cask install virtualbox-extension-pack
```

### Installing Vagrant

```
brew cask install vagrant
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


