==================================================================
“Neutron is not responding Error” on Openstack Web user interface
==================================================================

########
Symptoms
########
If you receive an error “Neutron is not responding Error” when listing the instances within a project, or that when looking at the “Networks” in the project, it is not populating the subnets that are part of the Network. On a command line, when running the command “openstack subnet list” the system returns with “An unknown error occurred”.
Explanation
This issue has been known to occur if a network or subnet or router has been deleted in a particular way, such that the “subnet” still exists, but has been disassociated from the “network”. This means that in the neutron database (on osdb1.nubes.rl.ac.uk), the subnet table has an entry where the “network_id” IS NULL value. This seems to break the Openstack web user interface.

########
Solution
########
Please contact cloud-support@stfc.ac.uk
