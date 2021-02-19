.. _flavors:

=============
Flavors
=============

The STFC Cloud provides a number of flavors to users but these fall into 3 categories at the moment.

Note: our flavors are created in so that they will efficiently pack onto the hardware we have - due to this we cannot create additional flavors as that may lead to scheduling difficulties and under utilisation.

################################
m*.* (e.g. m1.small or m3.large)
################################
.. Warning:: Deprecated

    Please use c* flavors instead

These are over committed flavors meaning that any VCPU core assigned to your VM may be shared with up to 3 other VMs.

This means that these VMs can be great for light workloads such as interactive testing or hosting web services but are not generally suitable for hosting services or doing intensive work.


################################
c*.* (e.g. c1.small or c3.large)
################################
These are flavors with dedicated CPU cores meaning that a VCPU core assigned to your VM is dedicated to you.

These flavors are great for computation and are optimised to either be within a single NUMA node providing great performance or balanced equally across the NUMA nodes in the hypervisor.

#######
l*.*
#######
These are flavors which utilize local SSD storage on the hypervisors with all of the storage being allocated to the root disk. These instances currently cannot be live migrated so in the event of hardware errors the machine will likely be lost or down for an extended period

Otherwise the flavors behave as c*.* flavors

#######
le*.*
#######
These are flavors which utilize local SSD storage on the hypervisors with 100GB being allocated to the root disk and the rest to an ephemeral disk. These instances currently cannot be live migrated so in the event of hardware errors the machine will likely be lost or down for an extended period.

Otherwise the flavors behave as c*.* flavors


#######
g*.*
#######
These have the same CPU configuration as c*.* flavors but have a GPU attached:

- g-k620.* have a NVidia Quadro K620 card attached
- g-p4000.* have one or more NVidia Quadro P4000 cards attached (the smallest has 1, the largest has 4)
- g-rtx4000.* (COMING SOON est May 2021) have one or more NVidia Quadro RTX 4000 cards attached (the smallest has 1, the largest has 4)
- g-p100.* have one or more NVidia Tesla P100 cards attached (the smallest has 1, the largest has 2)
- g-v100.* have one or more NVidia Tesla V100 32GB cards attached (small has 1,medium has 2 and large has 4)
- g-a100-40gb.* (COMING SOON est May 2021) have one or more NVidia Tesla A100 40GB) cards attached (the smallest has 1, the largest has 4)
