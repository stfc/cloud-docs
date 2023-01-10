================
Common Commands
================

On Train, Hypervisors often disable themselves and remove their services from the pool of resources if
there are 10 build failures in a row on a hypervisor.

#######################
List available images
#######################

To list available images do:

.. code-block:: bash

  openstack image list

#############################
List available flavors
#############################

To list available flavors run:

.. code-block:: bash

  openstack flavor list


############################################
List available networks
############################################

To list available networks run:

.. code-block:: bash

  openstack network list

############################################
List running servers
############################################

Showing all hosts in your current projects:

.. code-block:: bash

  openstack server list

######################
Show a specific server
######################
Looking at a specific server:-

.. code-block:: bash

  openstack server show <server ID>
