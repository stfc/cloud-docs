==============================================================================
Process for fixing issue where VM hosts are uncontactable from rest of network
==============================================================================

| Martin Summers 
| Version1 
| 2018-03-08 

########
Symptoms
########

Users lose connectivity to their virtual machines hosted on our cloud system “https://Openstack.stfc.ac.uk” and cannot
“ping” the Virtual machines or connect to them via SSH or any other services the VM’s have running on them. It is still
possible to connect to the hypervisor, and even start a console access to the VM itself, but logging into it is
difficult if the login is a non-local user to the VM.

######
Cause
######

We believe that the Virtual machines are no longer able to send reply packets back through the “correct” default
gateway interface (which at the time of writing is using a virtual IP address for the default gateway as “asymmetric
routing for vxLANs” is being implemented), and this shows the symptom of the VM’s being uncontactable.

########
Solution
########

Using “Symmetric vxLAN routing” should get round this issue, but if still using “Asymmetric vxLAN routing, then the
following procedure is recommended: 

1) Login to web Openstack page as admin (https://Openstack.stfc.ac.uk ) 

Find the VM(s) that can no longer be contacted and work out which Hypervisor it is running on. Hint: hostnames of
hypervisors usually start with the letters HV. Worth noting that the IP address network for VMs is 130.246.212.0 /22 –
which covers 130.246.212.1 to 130.246.215.254 IP addresses. 
Chances are that all the VM hosts on that hypervisor cannot be contacted. Can affect more 
than one Hypervisor - so be aware. Suggest attempting to ping other VM’s on other Hypervisors that are in other Racks –
as HV’s (to date) usually uplink to the router/switch in the same rack.


2) Go into Mimic and find Hypervisor. Select “Openstack” or “OpenNebula” under cloud on the Left Hand Side.
Suggest using browser find function to point to “compute” functions on Right Hand Side display, and use mouse
to hover over boxes under each heading that has “compute” in it – until the right hypervisor has been found.
Once found, click on the box. A pop-up window appears:- 


.. image:: /assets/faultfixes/CloudVMHostUncontactable/hypervisor_networking.png
    :width: 603px
    :height: 108px
    :align: center
    :alt: hypervisor network diagram


Above shows hv50 hypervisor uplink connected to swt-sn2100-03 router – which is the cumulus router in that rack that
hv50 depends upon. This will show which top of rack router the Hypervisor is connected to.

3) Find the admin 10.5.x.y IP address of the switch in question.
For example, look on the URL http://www-private.gridpp.rl.ac.uk/network/?subnet=10.5.0.0%2F24 

.. image:: /assets/faultfixes/CloudVMHostUncontactable/hypervisor_network_table.png
    :width: 608px
    :height: 276px
    :align: center
    :alt: network table

4) ssh to the switch management IP found in the previous step, using openVPN client. (need root password of
switch to login to this). (Note: details of installing OpenVPN can be found on :- https://wiki.e-science.cclrc.ac.uk/web1/bin/view/EScienceInternal/OpenVPNWindows
)
5) Find the interface (VLAN definition)  that the VIP(Virtual IP address)  is bound to in the interface config (/etc/network/networks.d/interfaces )– usually IP address 130.246.212.1 (for example). The interface has 3 or 4
settings that need to be commented out – the lines that begin with the word “address”. Comment out using a “#”
character at the beginning of each line.  Save the settings, then reload the interface settings using the command
“ifreload –a” (this disables the interface). Uncomment the lines previously commented out earlier,
and reload again - the interface should become active. At this stage the VMs should be contactable again.
The interface changes can be seen in Observium as a “port deleted”

.. image:: /assets/faultfixes/CloudVMHostUncontactable/eventlog.png
    :width: 519px
    :height: 237px
    :align: center
    :alt: network table
