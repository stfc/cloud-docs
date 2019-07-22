==============================
FAQS
==============================
updated:2018-09-05
version 0.2

##################################################
How do I get a new project in the STFC Cloud
##################################################
Suggest that as a bare minimum, the following are specified:-
1) Project name (what you want your project to be called)
2) Project description (a brief description of the project - not essential, but useful if other similar named projects)
3) Users - their full names, and e-mail addresses. If they have internal STFC Federal ID logins, these are useful too.
4) Types of machines you use and how many (This is so that we can assign a resource quota for the project, how many CPUs, memory, disk space .etc.)
5) If the project either now, or in the known future, needs to host internet accessible services - what those services are, their purpose and how many IP addresses that are externally facing will be required.

##################################################
How do I get support for the STFC cloud?
##################################################
E-mail the Cloud support team on  cloud-support@gridpp.rl.ac.uk , cloud-support@gridpp.rl.ac.uk.
There is no telephone support number at this time.

##########################################
How do I get access to the cloud?
##########################################
Contact the Cloud support team on cloud-support@gridpp.rl.ac.uk for a user logon account and a default project. Once you have these, you
can connect using http://openstack.stfc.ac.uk (for advanced users),or https://cloud.stfc.ac.uk for a more basic user interface.

##########################################################
Which Web browsers are supported for the web user interfaces ?
##########################################################
Recommended to use the latest Firefox or Google Chrome browser (date: 2018-09-05: Firefox version 61.0.2, Chrome version 68.0.3440.106)

#####################################
How do I connect to my VM using ssh?
#####################################
By default VMs in have ssh open from within STFC, If you are external we will usually open ssh to one of your floating IP addresses as part of the setup of your project. You can then ssh into this machine using your client of choice.

########################################################
How do I copy files to and from my VM and between VM's?
########################################################
The simplest way is with SCP, this is a default tool within linux distributions and various versions are available for Windows.

#############################################################
How do I run a command on lots of my VM's at the same time?
#############################################################
ClusterSSH is a good way of doing this and is pretty straightforward if you are using a linux desktop. Under Windows it is more complicated.

###########################################################################################
When I use "sudo" command on a VM I created, it says <my username> not in Sudoers - Help!
###########################################################################################
Occasionally this happens to a VM if something goes wrong or is too slow during the boot process.
In most cases a reboot of the VM from within OpenStack will fix this.
If it doesnt please email cloud-support@gridpp.rl.ac.uk

#####################################################
How do I get a remote X session displayed from a VM?
#####################################################
Documentation coming soon.

################################################################################
How do I get a remote linux desktop GUI display on my VM on my windows desktop?
################################################################################
If you have a GUI installed and running on your VM then you can use the console view through the https://openstack.stfc.ac.uk (or https://cloud.stfc.ac.uk within STFC).
Otherwise you can install a vncserver and VNC client to view.

#######################################################################################################
How can I view an Application on my Linux VM that requires some graphics acceleration and uses OpenGL?
#######################################################################################################

###########################################################
How do I create a VM that has GPU capability in my project?
###########################################################
If you have g* flavors available in your project then you can create a GPU accelerated VM.
If you dont then you can contact cloud-support@gridpp.rl.ac.uk to request access.

###########################################################################
How do I create a VM with shared disk space to my other VMs in my project?
###########################################################################
Documentation coming soon.

#####################################################################
How do I connect to another VM in my project which I did not create?
#####################################################################
In most cases we expect users to provide access to VMs to the other members of their project as required.
If this hasnt been possible then please contact cloud-support@gridpp.rl.ac.uk and we can help with this.


########################################################
What do I do if a new VM that I create does not work?
########################################################

#########################################################
How do I access my Virtual machines across the internet?
#########################################################
You will need to contact cloud-support@gridpp.rl.ac.uk to request a firewall hole for your floating IP.
We will then conduct some security checks and help you through this process

##########################################################################
Is there a default firewall policy on my project (and what does it say ?)
##########################################################################
Documentation coming soon.

###########################################################################################
I have a Database listening on its defaut port, but I can't connect to it remotely - Why?
###########################################################################################
Talks about default policy and adding one-off "additonal" policies for specific hosts

#######################################################################
A VM machine I was using has died  - How can I get the data off of it?
#######################################################################
Depending on the way the VM has failed we may be able to help get this back. Contact us at cloud-support@gridpp.rl.ac.uk

#######################################################################################################
What are the current machine "flavors"? Can I have  a machine that looks like a flavor you don't have?
#######################################################################################################
Show the different Flavor types available, and answer how users can create their own local project flavours if needed.

###################################################
Can you provide operating system "X" on Openstack?
###################################################
Policy doc on how we deal with new OS requests

###############################################
What sort of CPU performance should I expect?
###############################################
This varies between flavors with c* flavors offering the best performance per core

###################################################################
What sort of disk I/O performance should I expect locally on a VM?
###################################################################
Instances are currently limited to 200 IOPS read and write.

########################################################
What sort of network bandwidth should I expect on a VM?
########################################################
Hypervisors are currently connected at either 10gb or 25gb so you can expect a share of this depending on the size of the VM and the contention on the host.

#############################################
Can you recover a VM I accidentally deleted?
#############################################
Unfortunately we cannot.

#########################################
How do I login to the "admin" interface?
#########################################
Visit https://openstack.stfc.ac.uk

#####################################################################
How do I obtain a host certificate for my Openstack virtual machine?
#####################################################################
No pre-created Host certificates - user has to obtain them from Cert site.

###################################################################
My host seems to have rebooted since last time I logged in - why?
###################################################################
This is rare but usually this is due to an issue when migrating a VM which has triggered a reboot.

########################################################
What are the default DNS servers for VMs on Openstack?
########################################################
By default VMs in OpenStack use the DHCP agents within their Project networks.
