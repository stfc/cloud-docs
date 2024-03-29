.. _flavors:

=============
Flavors
=============

The STFC Cloud provides a number of flavors to users but these fall into 4 categories at the moment.

No flavor has resilient storage. If this is required you should boot from Volume at VM creation time

Note: our flavors are created in so that they will efficiently pack onto the hardware we have - due to this we cannot create additional flavors as that may lead to scheduling difficulties and under utilisation.

################################
c*.* (e.g. c1.small or c3.large)
################################

.. Warning:: C-flavors

    C flavors are now deprecated and new VMs will not be able to be created after the 12th of April 2023. It is recommend to use l flavors instead.

These are flavors with dedicated CPU cores meaning that a VCPU core assigned to your VM is dedicated to you.

These flavors are great for computation and are optimised to either be within a single NUMA node providing great performance or balanced equally across the NUMA nodes in the hypervisor.

#######
l*.*
#######
These are flavors which utilize local SSD storage on the hypervisors with all of the storage being allocated to the root disk. These instances currently cannot be live migrated so in the event of hardware errors the machine will likely be lost or down for an extended period.

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

- g-p4000.* have one or more NVidia Quadro P4000 cards attached (the smallest has 1, the largest has 4)
- g-rtx4000.* have one or more NVidia Quadro RTX 4000 cards attached (the smallest has 1, the largest has 4)
- g-rtx4000-ref.* have one or more NVidia Quadro RTX 4000 cards attached (the smallest has 1, the largest has 4)
    - these are old out of warranty nodes which have been refitted with new GPUs. For this reason these are reserved for short term use such as training courses. If you have a use case you want to use these for then please get in touch with support.
- g-a4000.* have one or more NVidia RTX A4000 cards attached (the smallest has 1, the largest has 4)
- g-a4000-ref.* have one or more NVidia RTX A4000 cards attached (the smallest has 1, the largest has 4)
    - these are old out of warranty nodes which have been refitted with new GPUs. For this reason these are reserved for short term use such as training courses. If you have a use case you want to use these for then please get in touch with support.
- g-p100.* have one or more NVidia Tesla P100 cards attached (the smallest has 1, the largest has 2)
- g-v100.* have one or more NVidia Tesla V100 32GB cards attached (small has 1,medium has 2 and large has 4)
- g-a100.* have one or more NVidia Tesla A100 40GB) cards attached (the smallest has 1, the largest has 4)
- g-a100-n100.* have one or more NVidia Tesla A100 40GB cards attached (the smallest has 1, the largest has 4) with higher core count and more network bandwidth
    - These are currently reserved for a specific project.
- g-a100-80gb.* have one or more NVidia Tesla A100 80GB cards attached (the smallest has 1, the largest has 8)
- g-arc-a770.* have one or two Intel Arc A770 16GB cards
    - These are very limited in number
- g-w6600.* have one or more AMD W6600 8GB cards (the smallest has 1, the largest has 4)
    - These are very limited in number
