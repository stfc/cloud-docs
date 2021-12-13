================================
Associate a floating IP to a VM
================================

Assumes that you have float-IPs associated with your project. If you do not - contact Cloud Support.

In the Horizon Web user interface (https://Openstack.stfc.ac.uk) in the Left Hand Side Menu, select:-

Compute->Instances

Find the VM (that has a Private 192.168.x.y IP address) that you wish to associate a Floating IP address with.
On the Right Hand side, click on the down arrow, and select "ASSOCIATE FLOATING IP" - a pop-up window will appear.
Click on "Select an IP address" - a drop-down list of available floating IP addresses will be listed. Click on one.
Click on the "ASSOCIATE" button. 
You should be returned to the "Instances" list, and you should now see your float-IP associated with your VM, as well as its original 192.168.x.y address.

For the equivalent using the openstack API command line:-

To find float-IP addresses associated with your project, use the command:-

openstack floating ip list --project "project name"

...where project name is your project name in quotations. 
This will return a list of float IP addresses, and will show which ones are in use, and which ones are not. Take note of one of the float IP addresses that is free.
Now list the VM instances in your project, use the command:-

openstack server list --project "project name"

....will list all of your VM instances. Take note of the first column, which is the instance ID of your chosen VM. To associate a floating IP selcted earlier, use the command:-

openstack server add floating ip <VM instance ID> <Floating IP>

This should then result with the floating IP address being associated with the floating IP names. You should now be able to ssh directly into the VM, using the floating IP address as a target, assuming that the security group allows ssh inbound, and any additional firewalls in the way, also have ssh allowed inbound. 


