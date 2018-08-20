==============================
FAQS
==============================



########################
Common Commands
########################

On Queens, Hypervisors often disable themselves and remove their services from the pool of resources if 
there are 10 build failures in a row on a hypervisor.

-----------------------
List Hypervisor Service
-----------------------

To list hypervisor services do:

.. code-block:: bash

  nova service-list

You will see output like below.

.. image:: /assets/faqs/list_hypervisor_output.png
    :width: 400px
    :height: 100px
    :align: center
    :alt: command line output


-----------------------------
Re-enable Hypervisor services
-----------------------------

To re-enable the service status on a hypervisor, run:

.. code-block:: bash

  nova service-enable <ID of hypervisor>

To batch enable all disabled hypervisors, something like:

.. code-block:: bash

  Nova service-list | grep –I disabled | awk -F\| '{ print $2 }' | sed 's/ //g' | xargs –i nova service-enable {}

This should re-enable each hypervisor one by one.

--------------------------------------------
Show all openstack hosts across all projects
--------------------------------------------

Showing all hosts in openstack across all projects:

.. code-block:: bash

  openstack server list --all-projects

----------------------
Show a specific server
----------------------
Looking at a specific server:-

.. code-block:: bash

  openstack server show <server ID>
