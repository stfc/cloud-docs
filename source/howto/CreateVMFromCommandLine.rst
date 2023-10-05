=============================================================
Create a Virtual Machine from command line in STFC Openstack
=============================================================

It’s common to want to create VMs from command line. To do this, you need to be on a host that has the openstack command line interface installed.

On our Ubuntu Focal 20.04 image the correct repository is already available, so you can use you install the client by running:

.. code-block:: bash

  sudo apt install python3-openstackclient



#############################################
Setting up the environment to select project
#############################################

After logging into a host, you have to set your environment variables.

The recommended method is downloading this script to set these up for you from: https://openstack.stfc.ac.uk/project/api_access/openrc/ (You will need to be signed in to the project you wish to use in the OpenStack web interface for this to work.)

Or set them yourself in a script:-

.. code-block:: bash

  export OS_AUTH_URL=https://openstack.stfc.ac.uk:5000/v3
  export OS_PROJECT_ID=<project ID>
  export OS_PROJECT_NAME=<project name>
  export OS_PROJECT_DOMAIN_NAME=<domain name>
  export OS_USER_DOMAIN_NAME=<user domain>
  export OS_PROJECT_DOMAIN_ID=<project domain ID>
  export OS_USERNAME=<fed ID>
  export OS_PASSWORD=<password>
  export OS_INTERFACE=public
  export OS_IDENTITY_API_VERSION=3

Please take note of the required details from openstack by going to the desired project, and going into the submenu API_ACCESS (https://openstack.stfc.ac.uk/project/api_access/) and click on view credentials.

.. csv-table:: variables
  :header: "Parameter", "Description"

  "OS_PROJECT_NAME", "they are the project that you wish to create the new VMs in."
  "OS_USERNAME and OS_PROJECT_DOMAIN_NAME", "Are the username and the domain that user is in, which you are using to create the VM."
  "OS_PASSWORD",  "Is your login to openstack (in plain text)." 

It is common to put all of the above in a shell file (just as above, but with a file extension of .sh) and then “source” the environment variables so that they are part of your command shell.

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

  openstack server create --flavor <flavor ID/name> --image <image ID/name> --network <network> --security-group default --key-name <key name> <server name>

To delete a host, you can use the command:-

.. code-block:: bash

  openstack server delete <instance id>

You can also run with --debug after the openstack command – this will give you a step by step commentary as to what is happening when creating a virtual machine.
For example:-

.. code-block:: bash

  openstack --debug server create --flavor <flavor ID/name> --image <image ID/name> --network <network> --security-group default --key-name <key name> <server name>
##########
References
##########

The following is a good generic guide:-
https://docs.openstack.org/install-guide/launch-instance-provider.html
