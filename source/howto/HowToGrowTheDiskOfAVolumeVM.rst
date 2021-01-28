======================================================================================================
How to create a clone machine (with bigger disk) of an Openstack VM which has a bootable Volume
======================================================================================================

##########
Scenario
##########
In the STFC cloud, if you created a new VM in instances, the default setting in the “Sources” was to create the VM with a new volume.
If you run out of disk space on the VM, you can usually just “resize” it to a new Flavor, and you will have the same VM, but with more space.
However, with Volumes, the system ignores the disk sizing aspect of the flavour. This document describes how to create a clone of the original VM and increase the disk size, so that space is available on the new VM.

###########
Method
###########
Check your Quotas for our project: Volume sizes, number of volumes, and number of volume snapshots are all parameters for which your project has a quota for, and there is a fair chance that what you have allocated may not be enough. Check with a cloud admin first before proceeding if you are unsure.
Make a snapshot of the volume attached to your VM
Identify the volume attached to the VM: Go to Computein the LHS menu, then click on instances – click on the Instance name of the Instance you wish to build a clone of. You will see a screen similar to the one in the screen shot – look at the bottom where it says “Volumes Attached”

The long hex number is the volume name.
Click on “Volumes” in the LHS menu, then “Volume”. There should be a volume with the name identified in the previous screenshot:-

Click on the arrow on the RHS of the volume, and select “Create snapshot”. You will see a pop-up window – give the snapshot a name and a description, then click on the button “CREATE VOLUME SNAPSHOT (FORCE)”. This action will create a snapshot of your volume in the Volumes-> Snapshots area.
Create a new volume from the snapshot and increase its size.
You if the snapshot was successful, you should see it in the list of volume snapshots:-

Click on the RHS button and select “Create Volume” – a pop-up window appears: Give the new volume a name and a description, but increase the size to be the size that you want it to be:-

Click on the “CREATE VOLUME”. After a short while, you should see the new volume appear in your list of Volumes:-

Create a new VM, but Source from new volume
Click on the Compute, then Instances menu on the LHS, Click on the “Launch Instance” button.
Fill in the Details section as normal.
Click on Source: Select “Volume” in the “Select Boot source” – the volume that was created in the previous step should be visible – select this volume:-

Select a flavour on the LHS: Note that the disk size of the flavour has no effect on the volume.
Select Networks and choose the network you wish to have the VM start up on.
At this point, all of the LHS options should not have any asterisk (*) characters next to them, which means all the mandatory data has been filled in. You can then click on the “Launch instance” option.
The system should boot up and appear in the list of Instances, and when you log into it, it should show the full disk size of the new volume:-

Logging into the new host, you can see the disk space now available:-

Things worthy of note when creating a VM from an existing volume
The flavour of the new VM does not affect the disk size – only the volume parameters does this !
Once the volume is attached to a VM, you cannot remove it: the only way to remove the VM host from the volume is if the instance is created, and when the VM was created, the setting for the volume to persist still exists. (See document “Useful command line notes” in the TWKI, which describes a “cinder force-delete <volume id>” workflow.
When creating a new instance from a volume, the volume may still have previous users created in the system and their home directories and they may have their details still in the /etc/sudoers.d/cloud file. However, you will not find that your FEDID has been added to the /etc/sudoers.d/cloud file.
