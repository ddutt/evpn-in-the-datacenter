# VX Simulation on Windows

This article is intended to show how to setup a simulation environment on a Windows laptop or server.  As an example we will leverage the existing [cldemo-vagrant](https://github.com/cumulusnetworks/cldemo-vagrant)  environment.  This demo environment will deploy Cumulus VX and Ubuntu servers in the following topology:

![Reference Topology](../cldemo_topology.png)

- [Table of contents](#)
	- [Installing Virtualbox](#installing-virtualbox)
	- [Installing Git](#installing-git)
	- [Installing Vagrant](#installing-vagrant)
	- [Deploy the cldemo-vagrant VMs](#deploy-the-cldemo-vagrant-vms)
	- [Manage the cldemo-vagrant VMs](#manage-the-cldemo-vagrant-vms)
		- [Connecting to the Console with the VirtualBox GUI](#connecting-to-the-console-with-the-virtualBox-gui)
		- [Connecting to the VMs with SSH](#connecting-to-the-vms-with-ssh)
	- [More about Vagrant](#more-about-vagrant)


## Prerequisites

 - Microsoft Windows Server 2012 R2+ or Microsoft Windows 10+
 - Hyper-V must be disabled*
 - SSH client (Putty, SecureCRT, etc)
 - Approximately 5GB disk space
 - Approximately 4GB free RAM

## Tools used

- VirtualBox [-Version 5.1.18 Installer-](http://download.virtualbox.org/virtualbox/5.1.18/VirtualBox-5.1.18-114002-Win.exe)  or [(Alternate version downloads)](https://www.virtualbox.org/wiki/Downloads)
- Git [-Version 2.12.2.2 Installer-](https://github.com/git-for-windows/git/releases/download/v2.12.2.windows.2/Git-2.12.2.2-64-bit.exe)
- Vagrant [-Version 2.0.2 Installer-](https://releases.hashicorp.com/vagrant/2.0.2/vagrant_2.0.2_x86_64.msi) or [(Alternate version downloads)](https://releases.hashicorp.com/vagrant/)





## Install the tools
Install VirtualBox, Git, and Vagrant tools on the machine.  This will require admin privileges on the server as well as a reboot.

### Installing Virtualbox

1. After downloading Virtualbox launch the installer.

![vbox_step1](./screenshots/vbox01.png?raw=true)

2. This guide will use the default installation with USB support, Virtual Networking, and Python support.

![vbox_step2](./screenshots/vbox02.png?raw=true)

3.  Virtualbox will install Virtual networking drivers and a new virtual ethernet adapter.

![vbox_step3](./screenshots/vbox04.png?raw=true)

4. Click through the next few screens with the defaults to start and complete the installation.

![vbox_step4](./screenshots/vbox07.png?raw=true)

VirtualBox should now be installed on the server and you can create/delete VMs on the new hypervisor.


### Installing Git

The git install includes the Git application but also installs a bash shell built on MINGW64 (Simiar to Cygwin) to provide Linux utilities on Windows.  It's a pretty great bash environment for Windows if you haven't used it before give it a try.  This guide however will concentrate on Windows deployments as such Powershell will be the primary method of interacting with Vagrant/Virtualbox.

 1. After downloading Git launch the installer.
 
![git_step1](./screenshots/git01.png?raw=true)
 
 2. Choose the install options in this guide we will enable large file support I also enable the associations for *.git and *.sh
 
![git_step2](./screenshots/git02.png?raw=true)

 3. Add Git to the windows PATH so that it's usable from both bash and Powershell.
 
![git_step3](./screenshots/git03.png?raw=true)

 4. Choose where to authenticate SSL certs when connecting to a Git repository.  If your company has an internal Stash, GitLab, or GitHub instance you might want to use your company's MS CA setup.  For this guide we are connected to public GitHub instances so the bundled certs and public authentication in OpenSSL work just fine.  If you're not sure choose the OpenSSL method, you can always change it later.

![git_step4](./screenshots/git04.png?raw=true)

 5. Choose how to checkout files from a Git repository.  Generally I like to make sure everything is committed in UNIX style line endings for maximum Windows/Linux compatibility.  Make the world a better place and stop the MS line endings.

![git_step5](./screenshots/git05.png?raw=true)

 6. Pick a terminal emulator for Git bash, optional really but I just default it to MinTTY

![git_step6](./screenshots/git06.png?raw=true)

 7. Couple more options.  Enable file system caching and Credential Manager.  Credential Manger basically saves you from having to type the same password for git commit every single time.  Have not messed with symbolic links on Windows.

![git_step7](./screenshots/git07.png?raw=true)

 8. Complete the installation across the next few screens.

![git_step8](./screenshots/git11.png?raw=true)


### Installing Vagrant

Vagrant is the final tool that will orchestrate all the VM creation and networking.  This install will require a reboot for both Server and Desktop versions of Windows.
 

 1. Install Vagrant from the msi installer.  

![git_step1](./screenshots/vagrant01.png?raw=true)

 2. Accept the terms and conditions as I'm sure you read all of them, correct?

![git_step2](./screenshots/vagrant02.png?raw=true)

 3. The usual choose your install directory, choosing the default here.

![git_step3](./screenshots/vagrant03.png?raw=true)

 4. Begin the installation it may prompt you for Admin privilege escalation during the install.

![git_step4](./screenshots/vagrant04.png?raw=true)

 5. Complete the install and reboot the machine.

![git_step5a](./screenshots/vagrant06.png?raw=true)

![git_step5b](./screenshots/vagrant07.png?raw=true)


## Setup the virtual topology
Time to actually do some networking, well virtual networking, OK fine it's more server stuff right now.

### Deploy the cldemo-vagrant VMs
 1. Launch PowerShell
 2. Clone the cldemo code locally with: `git clone https://github.com/CumulusNetworks/cldemo-vagrant.git`

![git_step2](./screenshots/ps01.png?raw=true)

 3. Change into the newly created cldemo-vagrant directory `cd cldemo-vagrant`

![git_step2](./screenshots/ps02.png?raw=true)

 4. Check the Vagrant status for the virtual machines with `vagrant status`

![git_step3](./screenshots/ps03.png?raw=true)

 5a. Install the Out-of-Band Server (jumpserver) virtual machine image with `vagrant box add CumulusCommunity/vx_oob_server --insecure --box-version=1.0.3 --provider virtualbox`

What happens here is that Vagrant will automatically detect that you do not locally have the VM you are trying to create so it will connect to the Vagrant Cloud image store and download.  This feature is one of the really powerful features of Vagrant as there are hundreds if not thousands of pre-built VMs, including Cumulus VX, available.  The oob-mgmt-server is built on a customized version of Cumulus VX.

 5b. Bring up your first VM the oob-mgmt-server with `vagrant up oob-mgmt-server`

Since this is the first time you bring up the VM the download may take a few mins to complete and then the demo sets up some tools on the server as part of the Vagrant setup.

 6. Once the Vagrant up completes, may take 5-10 mins, check the status of the VM with `vagrant status`

![git_step6](./screenshots/ps06.png?raw=true)

 7a. Install the Cumulus Vx image with `vagrant box add CumulusCommunity/cumulus-vx --insecure --box-version=3.3.2 --provider virtualbox`
 
 7b. Now bring up the oob-mgmt-switch with `vagrant up oob-mgmt-switch`

This step is very similar to step 5a in that Vagrant detects that the Cumulus VX image is not installed locally so it fetches the VM and installs it. 

 8. Finally once the oob-mgmt-switch has completed let's bring up some more nodes in the network: `vagrant up server01 leaf01 leaf02 spine01 spine02`

![git_step8](./screenshots/ps08.png?raw=true)


### Manage the cldemo-vagrant VMs

Now that we've deployed the VMs we can get to the actual networking fun.  For this we have a couple options using the VirtualBox GUI or ssh directly into the environment.

#### Connecting to the Console with the VirtualBox GUI

 1. First thing let's start with the VirtualBox GUI which basically gives a console into each VM.  Launch the Oracle VM VirtualBox application and you should see all the VMs you created with Vagrant.

![git_step1](./screenshots/vbox_gui01.png?raw=true)

 2. Then right click on the oob-mgmt-server and select "Show"

![git_step2](./screenshots/vbox_gui02.png?raw=true)

 3. You'll be connected to the console and it will ask about mouse capture.  The release key sequence is right control.  You will then be dropped onto the console of the oob-mgmt-server
  
![git_step3](./screenshots/vbox_gui03.png?raw=true)

 4. The default login for the oob-mgmt-server is username: cumulus password: CumulusLinux!

![git_step4](./screenshots/vbox_gui04.png?raw=true)

 5. From the oob-mgmt-server you should then be able to ssh into all of the devices you brought up.  (The cldemo pre-installs ssh keys for you)  For example `ssh leaf01`

![git_step5](./screenshots/vbox_gui05.png?raw=true)

#### Connecting to the VMs with SSH

 1. Vagrant spins up each VM with a port forward for SSH for each VM you bring up.  To find the port number that has been created use `vagrant port oob-mgmt-server`

![git_step1](./screenshots/putty01.png?raw=true)
 
 2. Launch putty and connect to the oob-mgmt-server 127.0.0.1:2222

![git_step2](./screenshots/putty02.png?raw=true)

![git_step3](./screenshots/putty03.png?raw=true)

### More about Vagrant

https://app.vagrantup.com/boxes/search

*Hyper-V locks VT-x/AMD-V on boot and therefore will not allow other hypervisors to leverage the hardware virtualization.  I will say it is possible to run both vBox and Hyper-V at the same time however if vBox tries to run a VM with VT-x support it will BSOD your server/workstation.  I'll write a followup article on using Hyper-V for your simulation environment once I get all the nested virtualization sorted.
