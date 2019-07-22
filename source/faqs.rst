==============================
FAQS
==============================
updated:2018-09-05
version 0.2

##################################################
Required Information for new Openstack Project
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
E-mail the Cloud support team on  cloud-support@helpdesk.gridpp.rl.ac.uk , cloud-support@gridpp.rl.ac.uk.
There is no telephone support number at this time.

##########################################
How do I get access to the cloud?
##########################################
Contact the Cloud support team on cloud-support@helpdesk.gridpp.rl.ac.uk for a user logon account and a default project. Once you have these, you
can connect using http://openstack.stfc.ac.uk (for advanced users),or https://cloud.stfc.ac.uk for a more basic user interface. 

##########################################################
Which Web browsers are supported for the web user interfaces ?
##########################################################
Recommended to use the latest Firefox or Google Chrome browser (date: 2018-09-05: Firefox version 61.0.2, Chrome version 68.0.3440.106)

#####################################
How do I connect to my VM using ssh?
#####################################

########################################################
How do I copy files to and from my VM and between VM's?	
########################################################

#############################################################
How do I run a command on lots of my VM's at the same time?	
#############################################################

###########################################################################################
When I use "sudo" command on a VM I created, it says <my username> not in Sudoers - Help!
###########################################################################################

#####################################################
How do I get a remote X session displayed from a VM?
#####################################################

################################################################################
How do I get a remote linux desktop GUI display on my VM on my windows desktop?
################################################################################
Suggest using vncserver and VNC client to view.

#######################################################################################################
How can I view an Application on my Linux VM that requires some graphics acceleration and uses OpenGL?
#######################################################################################################

###########################################################
How do I create a VM that has GPU capability in my project?	
###########################################################

###########################################################################
How do I create a VM with shared disk space to my other VMs in my project?
###########################################################################

#####################################################################
How do I connect to another VM in my project which I did not create?
#####################################################################

########################################################
What do I do if a new VM that I create does not work?
########################################################

#########################################################
How do I access my Virtual machines across the internet?	
#########################################################

##############################
How do I setup a new project?	
##############################

##########################################################################
Is there a default firewall policy on my project (and what does it say ?)
##########################################################################

###########################################################################################
I have a Database listening on its defaut port, but I can't connect to it remotely - Why?
###########################################################################################
Talks about default policy and adding one-off "additonal" policies forspecific hosts

#######################################################################
A VM machine I was using has died  - How can I get the data off of it?
#######################################################################
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
Shows latest benchmarks for CPU and when they were run,compared to known native systems.

###################################################################
What sort of disk I/O performance should I expect locally on a VM?
###################################################################
Disk I/O sample benchmarks.

########################################################
What sort of network bandwidth should I expect on a VM?
########################################################

#############################################
Can you recover a VM I accidentally deleted?
#############################################

###################################################################
Is there a migration path from the "Old cloud" to the "New cloud"?
###################################################################

#########################################
How do I login to the "admin" interface?
#########################################

#####################################################################
How do I obtain a host certificate for my Openstack virtual machine?
#####################################################################
No pre-created Host certificates - user has to obtain them from Cert site.

###################################################################
My host seems to have rebooted since last time I logged in - why?
###################################################################

########################################################
What are the default DNS servers for VMs on Openstack?	
########################################################

####################################################################
How do I find out what my "port usage" is against my Project Quota?
####################################################################
Describe methods of finding out how many ports are in use by a project vs the allocated quota.

