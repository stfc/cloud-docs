=============================================================
Create a Virtual Machine from command line in STFC Openstack
=============================================================

It’s common to want to create VMs from command line. To do this, you need to be on a host that has the openstack command line interface installed.

On Ubuntu machines, we can simply install the client by running:

.. code-block:: bash

  sudo apt install python3-openstackclient



#############################################
Setting up the environment to select project
#############################################

After logging into a host, you have to set your environment variables.

Please download the openrc script to set these up for you, from: https://openstack.stfc.ac.uk/project/api_access/openrc/ 

You will need to be signed in to the project you wish to use in the OpenStack web interface for this to work.

####################################
Finding images
####################################
A full detailed list of images is available on our Confluence: `Image Types
<https://stfc.atlassian.net/l/cp/KQ01NgEr/>`_.

Current images can also be listed on the command line with openstack. Run:-

.. code-block:: bash

  openstack image list

.. code-block:: bash

####################################
Finding flavors
####################################
A full detailed list of flavors is available on our Confluence: `Flavors
<https://stfc.atlassian.net/wiki/spaces/CLOUDKB/pages/211779756/Flavors>`_.

Current flavors can also be listed on the command line with openstack. Run:-

.. code-block:: bash

  openstack flavor list

.. code-block:: bash

####################################
Networks
####################################

To see the list of networks available, run the command:-

.. code-block:: bash

  openstack network list

…this returns two networks named “External” and “Internal”. Since we can’t add VMs directly to External network, we will be using the “Internal” network.

######################################################
Putting it all together to create a new Instance
######################################################
Please see the openstack documentation for the full set of parameters: https://docs.openstack.org/python-openstackclient/latest/cli/command-objects/server.html#server-create

.. code-block:: bash

  openstack server create

.. code-block:: bash

Here is an example command, putting together information from the previous commands:-

.. code-block:: bash

  openstack server create --flavor l3.nano --image Ubuntu-22.04 --network Internal --security-group default --key-name key_name the_servers_name

To delete a host, you can use the command:-

.. code-block:: bash

  openstack server delete <instance id>

You can also run with --debug after the openstack command – this will give you a step by step commentary as to what is happening when creating a virtual machine.
For example:-

.. code-block:: bash

  openstack --debug server create --flavor l3.nano --image Ubuntu-22.04 --network Internal --security-group default --key-name key_name the_servers_name

##########
References
##########

The following is a good generic guide:-
https://docs.openstack.org/install-guide/launch-instance-provider.html
