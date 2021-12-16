==============================
FAQS
==============================

.. contents::

.. _new_project:

##################################################
How do I get a new project in the STFC Cloud
##################################################
As a bare minimum, the following should be specified:-

1. Project name (what you want your project to be called)
2. Project description (a brief description of the project - not essential, but useful if other similar named projects)
3. Users - their full names, and e-mail addresses. If they have internal STFC Federal ID logins, these are useful too.
4. Types of machines you use and how many (This is so that we can assign a resource quota for the project, how many CPUs, memory, disk space .etc.)
5. If the project either now, or in the known future, needs to host internet accessible services - what those services are, their purpose and how many IP addresses that are externally facing will be required.

##################################################
How do I get support for the STFC cloud?
##################################################
E-mail the Cloud support team on cloud-support@gridpp.rl.ac.uk .

There is no telephone support number at this time.

.. _add_access:

##########################################
How do I get access to the cloud?
##########################################
Contact the Cloud support team on cloud-support@gridpp.rl.ac.uk . For a new project see :ref:`new_project`. 

If this is about an existing project please email with the project and account name you would like to add access for. 

Once you have these, you can connect using https://cloud.stfc.ac.uk for a simple user interface, or http://openstack.stfc.ac.uk for advanced users.

##############################################################
Which Web browsers are supported for the web user interfaces ?
##############################################################
The latest Firefox or Google Chrome browsers are recommended.

#####################################
How do I connect to my VM using ssh?
#####################################
By default VMs will be ssh-accessible from within the STFC network.

If you are external to STFC we will usually open ssh to one of your floating IP addresses whilst setting up your project.

If you are prompted for the password, ensure that your `private key is available <https://www.ssh.com/ssh/agent#adding-ssh-keys-to-the-agent>`_ 
and that you have used the right username. For example `ubuntu@<your_ip>` on Ubuntu instances.

########################################################
How do I copy files to and from my VM and between VM's?
########################################################
The simplest way is with SCP, this is a default tool within Linux distributions and various versions are available for Windows.

Here are two examples of copying a file from your home directory to /tmp would be:

.. code::

    scp ~/my_file.txt <username>@<ip_addr>:/tmp

.. code::

    scp ~/my_file.txt ubuntu@example.com:/tmp/my_renamed_file.txt

#############################################################
How do I run a command on lots of my VM's at the same time?
#############################################################
ClusterSSH is a good way of doing this and is pretty straightforward if you are using a Linux desktop. Under Windows it is more complicated.

###########################################################################################
When I use "sudo" command on a VM I created, it says <my username> not in Sudoers - Help!
###########################################################################################
Occasionally this happens to a VM if something goes wrong or is too slow during the boot process.

In most cases a reboot of the VM from within OpenStack will fix this.
If it doesn't please email cloud-support@gridpp.rl.ac.uk

#####################################################
How do I get a remote X session displayed from a VM?
#####################################################

This is not recommended for systems where untrusted parties have SSH access.

Enabling X forwarding comes with additional security considerations,
as authentication spoofing and verification attacks can occur.
`See the manual for additional details <https://man.openbsd.org/OpenBSD-current/man5/sshd_config.5#X11Forwarding>`_

X11 Forwarding must be enabled on the remote server first. In `/etc/ssh/sshd_config` 
set `X11Forwarding Yes`, if it is not already enabled and restart the sshd service.

To enable X11 forwarding within the SSH session, simply add `-X` to your SSH command. 
`-C` is highly recommended for traffic routed via external networks. Support for forwarding
is application specific, but most should work.

The following would enable compression and X11 forwarding:

.. code::

    ssh -CX foo@example.com

################################################################################
How do I get a remote linux desktop GUI display on my VM on my windows desktop?
################################################################################
If you have a GUI installed and running on your VM then you can use the console view:

- Navigate to https://openstack.stfc.ac.uk (or https://cloud.stfc.ac.uk within STFC)
- Click the dropdown next to the instance you would like to view
- Select console to view the video output

Otherwise you can install a vncserver and VNC client to view.

#######################################################################################################
How can I view an Application on my Linux VM that requires some graphics acceleration and uses OpenGL?
#######################################################################################################
Please contact us for further support.

#########################################################
How do I create a VM with GPU capabilities in my project?
#########################################################
If you have g* flavors available in your project then you can create a GPU accelerated VM. 
See :ref:`flavors for details on the GPU types available<flavors>`.

If you don't have g* flavors then you can contact cloud-support@gridpp.rl.ac.uk to request access.

###########################################################################
How do I create a VM with shared disk space to my other VMs in my project?
###########################################################################
Documentation coming soon.

#####################################################################
How do I connect to another VM in my project which I did not create?
#####################################################################
In most cases we expect users to configure access to their VMs as required.
If this hasn't been possible then please contact cloud-support@gridpp.rl.ac.uk and we can help with this.

For adding members to a project, so that they can modify cloud resources see :ref:`add_access`

########################################################
What do I do if a new VM that I create does not work?
########################################################
Please contact us as cloud-support@gridpp.rl.ac.uk 

.. _firewall_exceptions:

#########################################################
How do I access my Virtual machines across the internet?
#########################################################
You will need to contact cloud-support@gridpp.rl.ac.uk to request a firewall hole for your floating IP.
We will then conduct some security checks and help you through this process.

Once the exception is added to your floating ip(s) you will need to add create and associate a security group with the 
exception to your instance:

To create a new security group with one or more ports:

- Open Network, Security Groups
- Create a new Security Group and enter a name (such as `HTTP + HTTPS`) 
- Click next, leave the egress rules (as this allows traffic out) and add a rule per port
- Ensure Ingress is selected, and enter the port number.

To associate a new or existing security group:

- Click the dropdown by instances
- Edit port security groups
- Select Edit Security Groups for the interface to add these exceptions to (almost every VM will only have one)
- Add and remove groups using the `+` and `-` operators

##########################################################################
Is there a default firewall policy on my project (and what does it say ?)
##########################################################################
All egress is enabled.

The following ingress is enabled on IPv4 by default:

- All ICMP Traffic
- TCP/22
- UDP/7777

To add additional firewall ports see :ref:`firewall_exceptions`.

######################################
How do I avoid Docker Hub Rate limits?
######################################

STFC Cloud has their own docker hub mirror which is available to machines on the internal network. In-depth information can be found at: :ref:`docker_mirror_guide`.

This is free, does not apply rate limits and faster than Docker Hub, as it pulls over the local network. In the event the mirror is unavailable the instance will automatically pull from Docker Hub Directly.

New instances will use the mirror by default. Existing instances will also receive this change over time, but require restarting the docker service or instance to apply the update (see :ref:`restart_docker_service`).

If you want to manually verify that an instance is using the Docker Hub mirror see :ref:`check_docker_hub_mirror`.

###########################################################################################
I have a Database listening on its default port, but I can't connect to it remotely - Why?
###########################################################################################
Databases are not externally accessible under default policy.

Contact us to discuss the possibility of adding one-off "additonal" policies for specific hosts.

#######################################################################
A VM machine I was using has died  - How can I get the data off of it?
#######################################################################
Depending on the way the VM has failed we may be able to help get this back. Contact us at cloud-support@gridpp.rl.ac.uk

#######################################################################################################
What are the current machine "flavors"? Can I have  a machine that looks like a flavor you don't have?
#######################################################################################################

For most people the pre-configured flavors should fit almost every workload.
See :ref:`flavors` for the types available. 

Please contact us if you need assistance creating your own local project flavors.

###################################################
Can you provide operating system "X" on Openstack?
###################################################
Policy doc on how we deal with new OS requests

###############################################
What sort of CPU performance should I expect?
###############################################
This varies between flavors depending on workload, with c* flavors generally offering the best performance per core.
For a full list of the available flavors see :ref:`flavors`.

If you have concerns or further queries please feel free to contact us.

###################################################################
What sort of disk I/O performance should I expect locally on a VM?
###################################################################
Instances are currently limited to 200 IOPS read and write. 

For reference throughput (note: not latency) is comparable to a 15K SAS disk, 
or double the speed of a typical hard drive found in a desktop.

########################################################
What sort of network bandwidth should I expect on a VM?
########################################################
Hypervisors are currently connected to either 10gb or 25gb links, so you can expect a share of this depending on the size of the VM 
and the number of VMs on the same host.

#############################################
Can you recover a VM I accidentally deleted?
#############################################
Unfortunately, we cannot.

If a volume was attached, and it was not selected during deletion process, it can
be attached to a new instance to access the data on it.

#########################################
How do I login to the "admin" interface?
#########################################
Visit https://openstack.stfc.ac.uk

#####################################################################
How do I obtain a host certificate for my Openstack virtual machine?
#####################################################################
We don't provide certificates. These can be issued from the internal certificate site,
an external vendor or be re-used from an existing deployment.

###################################################################
My host seems to have rebooted since last time I logged in - why?
###################################################################
This is rare, but usually this is due to an issue when migrating a VM which has triggered a reboot.

If you have concerns please feel free to contact us for additional support.

########################################################
What are the default DNS servers for VMs on Openstack?
########################################################
By default VMs in OpenStack use the DHCP agents within their Project networks as
the DNS server IPs can change. 

The current DNS server IPs can be viewed by viewing the following file on non-Systemd machines (such as SL7):

.. code::

  cat /etc/resolv.conf

For Systemd based machines (such as Ubuntu) the current DNS servers can be viewed by doing:

.. code::

  systemd-resolve --status
