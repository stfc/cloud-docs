===============================================
How to Create a Virtual Machine from command line in STFC Openstack
===============================================

It’s common to want to create VMs from command line. To do this, you need to be on a hot that has the openstack command line interface installed. Common examples are keystone1.nubes.rl.ac.uk, nova1.nubes.rl.ac.uk. It could be any of the infrastructure hosts.
#########
Setting up the environment to select project
#########
After logging into a host, you have to set your environment variables – preferably in a script:-

.. code-block:: bash

  export OS_AUTH_URL=https://openstack.nubes.rl.ac.uk:5000/v3
  export OS_IDENTITY_API_VERSION=3
  export OS_PASSWORD=************
  export OS_PROJECT_DOMAIN_NAME=default
  export OS_PROJECT_NAME="SCD Cloud Operations Team"
  export OS_TENANT_NAME="SCD Cloud Operations Team"
  export OS_USERNAME=<username>
  export OS_USER_DOMAIN_NAME=<userdomain>

Take note of the “PROJECT-NAME” and “TENANT-NAME” – they are the project that you wish to create the new VMs in.  The USER_DOMAIN_NAME and USERNAME are the user and the domain that user is in, that you are using to create the VMs. PASSWORD is your login to openstack – in clear text.
It is common to put all of the above in a shell file (just as above, but with a file extension of .sh) and then “source” the environment variables so that they are part of your command shell So, if I put the above list of commands in a file “martin_openstack.sh” then I need to run “source ./martin_openstack.sh” after I have logged into a openstack server.
#########
Finding images and flavors available
#########
Once on the command line of a server, you need to list the flavours available for the project to use as well as the images available.
To see the images available, run:-

.. code-block:: bash

  openstack flavor list


.. csv-table:: flavor
  :header: "ID", "Name","RAM","Disk","Ephemeral","VCPUs","Is Public"

  "026ace2c-5247-4bdc-8929-81d129cc69bf","c3.small","4096","20","0","2","True"
  "1","m1.tiny","1024","10","0","1","True"
  "11","c1.xlarge","16384","100","0","4","True"
  "110dd30b-3d5f-4a4a-b380-ba6dfcfa5ad3","c2.large","8192","80","0","8","True"
  "12","c1.xxlarge","32768","160","0","8","True"
  "15bb0664-ffc8-45e4-9a99-06f3fcfcc680","m2.xlarge","16384","160","0","16","True"
  "2","m1.small","2048","20","0","1","True"
  "20c58ba0-f87c-423d-9ff1-2c7ae1bef7cc","c3.xlarge","32768","160","0","16","True"
  "26a01e1d-2e32-4949-a390-d2e6383d2ae7","c1.3xl","49152","200","0","16","True"
  "3","m1.medium","4096","40","0","1","True"
  "34900a49-ad26-490d-95f5-87aeb6c12d7c","m3.xlarge","32768","160","0","16","True"
  "37c83b1b-05ab-4169-9022-78b30f5450d8","m3.small","4096","20","0","2","True"
  "4","m1.large","8192","80","0","2","True"
  "45f004b0-97e9-403b-a63a-90ed9af70087","g-k620.xxlarge","32768","160","0","8","False"
  "5","m1.xlarge","16384","160","0","8","True"
  "52786a6d-e20a-4906-82db-4ce1d91a715f","m2.small","2048","20","0","2","True"
  "6cf0813c-1ba2-4999-b7eb-34d71a2a4199","c3.medium","8192","40","0","4","True"
  "6e6062f3-4744-4320-8dc3-767795ec98e8","m2.large","8192","80","0","8","True"
  "6f5c0478-84c2-4478-a6d5-073d97a81c0e","c3.large","16384","80","0","8","True"
  "78e477cc-a3e6-4ec8-af99-c59f33f02c3c","g-k620.tiny","1024","10","0","1","False"
  "7ca6ba0f-416a-4a8c-87a7-0f2890c9fd11","m1.test","1024","10","0","1","False"
  "7ff4bca9-2e13-42e0-9283-cf17cff372f3","c2.small","2048","20","0","2","True"
  "8","c1.medium","4096","40","0","2","True"
  "9","c1.large","8192","80","0","2","True"
  "a033c03d-e684-47a7-be9f-a857de135c4c","c2.xlarge","16384","160","0","16","True"
  "a7716bcf-490d-4c01-a518-b25587cc02e8","m3.large","16384","80","0","8","True"
  "bcea5cd1-ccc1-45aa-a771-82cf2deb41ba","c2.medium","4096","40","0","4","True"
  "c7ee6c89-3059-4bc1-b332-317bdcb4da36","m3.medium","8192","40","0","4","True"
  "ce0828cb-132c-4890-8b78-c7c123804e43","c1.4xl","92160","400","0","28","True"
  "d0184b50-bce2-4679-9b00-c1b774f9c647","m3.tiny","2048","20","0","1","True"
  "e166d59d-fab6-4839-9f04-ca4b275262c3","g-k620.4xl","128000","400","0","30","False"
  "faa9265d-98e4-4cc6-acd7-fa8a7e72e8ef","m1.xxlarge","32768","160","0","8","False"
  "fc04f5fc-c264-4aa9-b1bf-fc3aa7736cbc","m2.medium","4096","40","0","4","True"

To see the choice of images available, run the command:-

.. code-block:: bash

  openstack image list

.. csv-table:: images
  :header: "ID","Name","Status"

  "b8c3c82e-1ba3-4c4e-9d09-eb713cbe52c6","Next3-ScientificLinux-7-Gui","active"
  "d3becd76-8046-4c9e-ab9d-e476b40237c7","ScientificLinux-6-AQ","active"
  "8ba8781a-87a9-4f11-ae57-3865c19e8be9","ScientificLinux-6-Gui","active"
  "1bda5d33-b718-4a0e-a330-037e6096bb9c","ScientificLinux-6-NoGui","active"
  "2e8fb278-c5d8-4647-b13c-e63c577fe4ae","ScientificLinux-7-AQ","active"
  "44aa5e0e-cf74-4e71-ab2c-b11cf5dd1e66","ScientificLinux-7-Gui","active"
  "3741c38f-f59a-4fd5-89b0-f61f2d577b23","ScientificLinux-7-NoGui","active"
  "5d8dfe3b-52e0-48e1-9219-88c47dbd8c8a","Ubuntu-Bionic-Gui","active"
  "02406ced-6980-4937-b9c5-38964cefd4d4","Ubuntu-Bionic-NoGui","active"
  "f29f4278-f168-489d-ae54-7aa1269755f2","Ubuntu-Trusty-Gui","active"
  "5a5178af-ef85-4184-bf4a-d607a43b248a","Ubuntu-Trusty-NoGui","active"
  "24cde165-b797-4fce-8322-59cd36dc596a","Ubuntu-Xenial-Gui","active"
  "e25b990f-8fd9-4a42-bf43-4d421f8e93e9","Ubuntu-Xenial-NoGui","active"
  "190cda0b-ac8e-42a9-af49-38484c88ac63","readthedocs_snapshot_2018-10-25","active"
  "147eefc8-ad2b-447f-8195-944fe4547ddd","xming_rdesktop_readthedocs_snapshot1","active"


To see the list of networks available, run the command:-

.. code-block:: bash

  openstack network list

…this returns two networks named “External” and “Internal”. Since we can’t add VMs directly to External network, we will be using the “Internal” network.
#########
Putting it all together to create a new Instance
#########

Here is an example command, putting together information from the previous commands:-

.. code-block:: bash

  openstack server create --flavor m1.tiny --image ScientificLinux-7-NoGui --nic net-id=Internal --security-group default --key-name xbe91637 test_2018-10-29_1511

…where flavour and image are from the previous commands used, net_id is the name of the Network to be used (note you can use the actual Net_ID number instead if preferred – it can make things faster!). Security group is defining the specific security group, and key-name, chooses the ssh keypair to include when creating the host. “test_2018-10-29_1511” is the name of the host that is being created – known within openstack.
Some useful extras
Adding --timing after the openstack command provides some statistics of how quickly various calls are being completed. You will see the usual host creation data, but at the end, you will also see the response times of each openstack API module.

.. code-block:: bash

  openstack --timing server create --flavor m1.tiny --image Ubuntu-Xenial-NoGui --nic net-id=Internal --security-group default --key-name xbe91637 test_2018-10-30_1357



To delete a host, you can use the command:-

.. code-block:: bash

  openstack server delete <instance id>

You can also run with --debug after the openstack command – this will give you a step by step commentary as to what is happening when creating a virtual machine.
For example:-

.. code-block:: bash

  openstack --debug server create --flavor m1.tiny --image Ubuntu-Xenial-NoGui --nic net-id=Internal --security-group default --key-name xbe91637 test_2018-10-30_1357

#########
References
#########

The following is a good generic guide:-
https://docs.openstack.org/mitaka/install-guide-ubuntu/launch-instance-provider.html
