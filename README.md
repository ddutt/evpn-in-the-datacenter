# EVPN in the Data Center 
![Reference Topology](./documentation/cldemo_topology.png "Reference Topology")


This repository contains configuration code with Ansible playbooks to deploy EVPN in the sample Clos topology shown in figure 6-1 of chapter 6 of the O'Reilly book [EVPN in the Data Center](https://cumulusnetworks.com/evpn-data-center-oreilly/).

Using this assumes two critical pieces of software are installed on your computer, specifically:
* [Vagrant](http://www.vagrantup.com)
* [Virtualbox](http://www.virtualbox.org) (or if you're on a Linux machine, [KVM](https://www.linux-kvm.org/page/Main_Page))

The section on [Prerequisies](#prerequisites-and-getting-started) describes how to install these pieces of software on your desktop/laptop. Running the entire simulation uses up at least 8GB of RAM on the machine it is running on. 

There are two demo options included in this repository:
* **[Centralized Routing](./ansible/cldemo-evpn-centralized)** -- This uses the exit leaves exit01/exit02 as EVPN routers while the leaves do EVPN bridging only
* **[Distributed Routing](./ansible/cldemo-evpn-distributed)** -- All the leaves do both EVPN routing and bridging, while the exit leaves provide the default route

**The individual demo pages contains a bunch of additional information such as descriptions of outputs and how to find information in FRR which is not included in the book.**

The Ansible playbooks are just glorified file copies. These playbooks are not meant to demonstrate network automation, but to keep things simple so that anyone can follow them. Basically, the playbooks copy the files provided under the config directory of each of the demos to the specific machines and reload interfaces and FRR. You can look at the configurations directly in the config directory for the specific machine without launching the VMs as well. Launching the VMs allows you to see the actual working of EVPN in a somewhat realistic, yet simple topology. You can also modify the configuration and see the effects of those changes. 

## Table of Conents

* [Quick Start](#quick-start)
* [Prerequisites and Getting Started](#prerequisites-and-getting-started)
  * [Windows](./documentation/windows)
  * [MacOS](./documentation/macos/)
  * [Linux (Ubuntu 16.04)](./documentation/linux)
* [Frequently Asked Questions](#frequently-asked-questions)
  * [What is Vagrant?](#what-is-vagrant)
  * [What is a Vagrantfile?](#what-is-a-vagrantfile)
  * [What is Libvirt/KVM?](#what-is-libvirtkvm)i
  * [What is Cumulus VX?](#what-is-cumulus-vx)
  * [Which Software Versions should I use?](#which-software-versions-should-i-use)
  * [What is the Out of Band Server Doing?](#what-is-the-out-of-band-server-doing)
  * [How are IP addresses Allocated?](#how-are-ip-addresses-allocated)
  * [Tips on Managing the VMs in the Topology](#tips-on-managing-the-vms-in-the-topology)
  * [Can I Preserve my Configuration?](#can-i-preserve-my-configuration)
  * [Switching Between the Demos](#switching-between-the-demos)
  * [Running More Than One Simulation at Once](#running-more-than-one-simulation-at-once)
  * [How Can I Customize the Topology?](#how-can-i-customize-the-topology)
  * [How Is This Different From Standard Cumulus Demos?](#how-is-this-different-from-standard-cumulus-demos)

## Quick Start:
If you've installed the appropiate software listed in the [pre-requisites](#prerequisites-and-getting-started) section, then:

**NOTE: On Windows, if you have HyperV enabled, you will need to disable it as it will
conflict with Virtualbox's ability to create 64-bit VMs.**

### Provision the Topology and Log-in

    git clone https://github.com/ddutt/evpn-in-the-datacenter.git
    cd evpn-in-the-datacenter
    vagrant up
    vagrant ssh
	su - cumulus
	cd ansible/cldemo-evpn-distributed
	ansible-playbook run-demo.yml
    ssh server01
	ping 10.1.3.103
	ping 10.2.4.104
	ping 10.0.0.253
	ping www.google.com

The pings are ordered to verify that:
* EVPN bridging works across racks
* EVPN RT-2 based routing works
* EVPN RT-5 routing works to reach the internet node
* EVPN RT-5 routing works to reach the Internet

### Small Topology

For a smaller topology, replace `vagrant up` with `vagrant up oob-mgmt-server oob-mgmt-switch leaf01 leaf02 spine01 spine02 exit01 server01 server02`. This takes up at least 4GB of RAN. 

You cannot ping the internet node or external nodes with this. But you can test centralized routing. 

## Prerequisites and Getting Started

### Instructions for:
* [Windows](./documentation/windows)
* [MacOS](./documentation/macos)
* [Linux (Ubuntu 16.04)](./documentation/linux/)

## Frequently Asked Questions

### What is Cumulus VX?
This repository makes use of [Cumulus VX](https://cumulusnetworks.com/cumulus-vx/) which is a virtual machine
produced by Cumulus Networks to simulate the user experience of configuring a switch using the Cumulus Linux network operating system.

### What is Vagrant?
[Vagrant](https://www.vagrantup.com/) is an open source tool for quickly
deploying large topologies of virtual machines. Vagrant and [Cumulus VX](#what-is-cumulus-vx) can be
used together to build virtual simulations of production networks to validate
configurations, develop automation code, and simulate failure scenarios.

Vagrant uses [Vagrantfiles](#what-is-a-vagrantfile) to represent the topology.

### What is a Vagrantfile?
Vagrant topologies are described in a text file called a "Vagrantfile," 
which is also the filename. A Vagrantfile is a Ruby program that
tells Vagrant which devices to create and how to configure their networks.
`vagrant up` will execute the Vagrantfile and create the reference topology
using Virtualbox. 

### What is Libvirt/KVM?
Libvirt/KVM is a high-performance hypervisor that is used on Linux systems **ONLY**.
[Vagrantfiles](#what-is-a-vagrantfile) for the Libvirt/KVM hypervisor are also included in this repository.
To use them you need to be using a Linux system and follow the [Linux setup instructions](./documentation/linux).

Libvirt/KVM offers several notable advantages over Virtualbox:
* There is no interface limit (virtualbox limits VMs to 36 interfaces)
* VMs can be started in Parallel which greatly reduces simulation startup time

As a result this tends to be the most common hypervisor for larger simulations.

### Which Software Versions Should I Use?
Software versions are always changing. At the time of this writing the following 
versions are known to work well: 
* Vagrant v2.0.2
* Virtualbox v5.1.22
* Libvirt v1.3.1

### What Is The Out Of Band Server Doing?
The following tasks are completed to make using the topology more convenient.

 * DHCP, DNS, and Apache are installed and configured on the oob-mgmt-server
 * Static MAC address entries are added to DHCP on the oob-mgmt-server for all devices
 * A bridge is created on the oob-mgmt-switch to connect all devices eth0 interfaces together
 * A private key for the Cumulus user is installed on the oob-mgmt-server
 * Public keys for the cumulus user are installed on all of the devices, allowing passwordless ssh
 * A NOPASSWD stanza is added for the cumulus user in the sudoers file of all devices

After the topology comes up, we use `vagrant ssh` to log in to the management
device and switch to the `cumulus` user. The `cumulus` user is able to access
other devices (leaf01, spine02) in the network using its SSH key, and has
passwordless sudo enabled on all devices to make it easy to run administrative
commands. Further, most automation tools (Ansible, Puppet, Chef) are run
from this management server. **Most demos assume that you are logged into
the out of band management server as the `cumulus` user**.

Note that due to the way we simulate the out of band network, it is not possible
to use `vagrant ssh` to access in-band devices like leaf01 and leaf02. These
devices **must** be accessed via the out-of-band management server.

### How are IP addresses Allocated?
The [Reference Topology](#what-is-the-reference-topology) only specifies the IP addresses used in the Out-of-Band network
for maximum flexibility when creating new demos. To see the IP address allocation for the
Out-of-Band Network check the [IPAM diagram](./documentation/ipam.md)

### Tips on Managing the VMs in the Topology
The topology built using this Vagrantfile does not support `vagrant halt` or
`vagrant resume` for in-band devices. To resume working with the demos at a later point in time, use 
the hypervisor's halt and resume functionality.

In Virtualbox this can be done inside of the GUI by powering off (and later powering-on) the devices 
involved in the simulation or by running the following CLI commands:

    * VBoxManage controlvm leaf01 poweroff
    * VBoxManage startvm leaf01 --type headless

When using the libvirt/kvm hypervisor the following commands can be used:

    * virsh destroy cldemo-vagrant_leaf01
    * virsh start cldemo-vagrant_leaf01


#### Factory-reset a device

    vagrant destroy -f leaf01
    vagrant up leaf01


#### Destroy the entire topology

    vagrant destroy -f

### Can I Preserve My Configuration
In order to keep your configuration across Vagrant sessions, you should either save your configuration
in a repository using an automation tool such as Ansible, Puppet, or Chef (preferred) or alternatively 
copy the configuration files off of the VMs before running the "vagrant destroy" command to remove and 
destroy the VMs involved in the simulation.

One helpful command for saving configuration from Cumulus devices is:

    net show configuration files

or 

    net show configuration command

**This command will not show configuration for third-party applications.**

### Switching Between the Demos
You can switch between the centralized and distributed demos in the same simulation. To do this, run the ansible playbook 'reset.yml' before running 'run-demo.yml'.

### Running More Than One Simulation At Once
Using this demo environment, it is possible to run multiple simulations at once. The procedure varies
slightly from hypervisor to hypervisor. 

#### Virtualbox
In the Vagrantfile built for Virtualbox there is a line which sets `simid= [some integer]` in order to
create unique simulations a text editor can be used to modify the simid value to something unique which 
does not match other running simulations on the simulation node.

#### Libvirt
In the Vagrantfile built for Libvirt (Vagrantfile-kvm) virtual networks are built from link to link
using UDP tunnels. In order to make sure that the VMs do not collide with each other. By default the
demo uses ports 8000-10000 but these values can be swapped either by:

**A).** Running the [Customize the Topology](#how-can-i-customize-the-topology)
workflow below and providing a '-s' argument
**OR**
**B).** By modifying the Vagrantfile-kvm directly, swapping the prepending 1000's place for the port numbers to something different
that do not overlap with any running applications or ports. In the example below we're swapping the ports used by the simulation
from 8000-10000 --> 30000-32000.
`_port => '8`  --> `_port => '30`
`_port => '9`  --> `_port => '31`


### How Can I Customize the Topology?

To create your own arbitrary topology, we recommend using Topology Converter. This will create a new 
Vagrantfile which is specific to your environment.For more details on how to make customized 
topologies, read Topology Converter's [documentation](https://github.com/CumulusNetworks/topology_converter/tree/master/documentation).

### How Is This Different From Standard Cumulus Demos?

This repository is based off of Cumulus cldemo-vagrant, but is not identical. Specifically, this is geared towards a simplified installation of the EVPN demo. The helper scripts have been modified for this specific use.

---

>©2018 Cumulus Networks. CUMULUS, the Cumulus Logo, CUMULUS NETWORKS, and the Rocket Turtle Logo 
(the “Marks”) are trademarks and service marks of Cumulus Networks, Inc. in the U.S. and other 
countries. You are not permitted to use the Marks without the prior written consent of Cumulus 
Networks. The registered trademark Linux® is used pursuant to a sublicense from LMI, the exclusive 
licensee of Linus Torvalds, owner of the mark on a world-wide basis. All other marks are used under 
fair use or license from their respective owners.

For further details please see: [cumulusnetworks.com](http://www.cumulusnetworks.com)
