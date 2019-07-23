=============
Flavors
=============

The STFC Cloud provides a number of flavors to users but these fall into 3 categories at the moment.

Note: our flavors are created in so that they will efficiently pack onto the hardware we have - due to this we cannot create additional flavors as that may lead to scheduling difficulties and under utilisation.

#######
m*.* (e.g. m1.small or m3.large)
#######
These are over committed flavors meaning that any VCPU core assigned to your VM may be shared with up to 3 other VMs.

This means that these VMs can be great for light workloads such as interactive testing or hosting web services but are not generally suitable for hosting services or doing intensive work.

#######
c*.* (e.g. c1.small or c3.large)
#######
These are flavors with dedicated CPU cores meaning that a VCPU core assigned to your VM is dedicated to you.

These flavors are great for computation and are optimised to either be within a single NUMA node providing great performance or balanced equally across the NUMA nodes in the hypervisor.

#######
g*.*
#######
These have the same CPU configuration as c*.* flavors but have a GPU attached:
- g-k620.* have a NVidia Quadro K620 card attached
- g-p4000.* have one or more NVidia Quadro P4000 cards attached (the smallest has 1, the largest has 4)
- g-p100.* have one or more NVidia Tesla P100 cards attached (the smallest has 1, the largest has 2)
