Instance Snapshot
==============================================================================================
A snapshot can be used as a **backup** or **template** for creating new instances.

.. contents:: Table of Contents

.. _Creating-Snapshot:

Creating Snapshots from Instance
--------------------------------
Web Interface
^^^^^^^^^^^^^
1. Go to :guilabel:`Compute` → :guilabel:`Instance` in Web Interface (https://openstack.stfc.ac.uk/)
2. Shut off your instance by clicking the drop-down menu on the right-hand side (in :guilabel:`Actions` column) and select :guilabel:`SHUT OFF INSTANCE`

.. image:: /assets/howtos/InstanceSnapshot/Create-Step2.png
    :align: center
    :alt:

3. Create a Snapshot by clicking the drop-down menu on the right-hand side (in :guilabel:`Actions` column) and select :guilabel:`CREATE SNAPSHOT`

.. image:: /assets/howtos/InstanceSnapshot/Create-Step3.png
    :align: center
    :alt:

4. Give it a name and click :guilabel:`CREATE SNAPSHOT`

.. image:: /assets/howtos/InstanceSnapshot/Create-Step4.png
    :align: center
    :alt:


Command-Line
^^^^^^^^^^^^^
See :ref:`Use-OpenStack-CLI` on how to set-up the command line client.

1. Find the server ID (Instance) using the command below

.. code-block:: bash

    $ openstack server list
    +--------------------------------------+--------------------------+---------+----------------------------------------+---------------------------------------------------------+--------------+
    | ID                                   | Name                     | Status  | Networks                               | Image                                                   | Flavor       |
    +--------------------------------------+--------------------------+---------+----------------------------------------+---------------------------------------------------------+--------------+
    | b5acb398-76b4-48fe-9b9a-d480636fdfd9 | test-rebuild             | ACTIVE  | Internal=172.16.101.195                | ubuntu-focal-20.04-gui                                  | c3.small     |
    +--------------------------------------+--------------------------+---------+----------------------------------------+---------------------------------------------------------+--------------+

2. Shut down the instance using

.. code-block:: bash

    $ openstack server stop <server-id>
    
    #example
    $ openstack server stop b5acb398-76b4-48fe-9b9a-d480636fdfd9

3. Confirm that the server is shut-off

.. code-block:: bash

    $ openstack server list
    +--------------------------------------+--------------------------+---------+----------------------------------------+---------------------------------------------------------+--------------+
    | ID                                   | Name                     | Status  | Networks                               | Image                                                   | Flavor       |
    +--------------------------------------+--------------------------+---------+----------------------------------------+---------------------------------------------------------+--------------+
    | b5acb398-76b4-48fe-9b9a-d480636fdfd9 | test-rebuild             | SHUTOFF | Internal=172.16.101.195                | ubuntu-focal-20.04-gui                                  | c3.small     |
    +--------------------------------------+--------------------------+---------+----------------------------------------+---------------------------------------------------------+--------------+

4. Use ``openstack server image create`` to create a snapshot

.. code-block:: bash

    $ openstack server image create --name test-snapshot b5acb398-76b4-48fe-9b9a-d480636fdfd9
    +------------+------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
    | Field      | Value                                                                                                                                                                                                                                                                                                                                                            |
    +------------+------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
    | created_at | 2021-12-03T12:37:40Z                                                                                                                                                                                                                                                                                                                                             |
    | file       | /v2/images/2b9c6711-4dd8-4e5c-9edc-dd106b8319b5/file                                                                                                                                                                                                                                                                                                             |
    | id         | 2b9c6711-4dd8-4e5c-9edc-dd106b8319b5                                                                                                                                                                                                                                                                                                                             |
    | min_disk   | 20                                                                                                                                                                                                                                                                                                                                                               |
    | min_ram    | 0                                                                                                                                                                                                                                                                                                                                                                |
    | name       | test-snapshot                                                                                                                                                                                                                                                                                                                                                    |
    | owner      | fa0f417fb4b5462791e4320e317eb2d2                                                                                                                                                                                                                                                                                                                                 |
    | properties | base_image_ref='90e1b77b-4192-46f1-8d9c-49fc36d9b54c', boot_roles='user', clean_attempts='2', description='Ubuntu-Focal-Gui', image_location='snapshot', image_state='available', image_type='snapshot', instance_uuid='b5acb398-76b4-48fe-9b9a-d480636fdfd9', locations='[]', os_distro='Ubuntu', os_hidden='False', os_variant='Gui', os_version='20.04-Focal' |
    | protected  | False                                                                                                                                                                                                                                                                                                                                                            |
    | schema     | /v2/schemas/image                                                                                                                                                                                                                                                                                                                                                |
    | status     | queued                                                                                                                                                                                                                                                                                                                                                           |
    | tags       |                                                                                                                                                                                                                                                                                                                                                                  |
    | updated_at | 2021-12-03T12:37:40Z                                                                                                                                                                                                                                                                                                                                             |
    | visibility | private                                                                                                                                                                                                                                                                                                                                                          |
    +------------+------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+

5. Check the image list

.. code-block:: bash

    openstack image list
    +--------------------------------------+----------------------------------------------------------+-------------+
    | ID                                   | Name                                                     | Status      |
    +--------------------------------------+----------------------------------------------------------+-------------+
    | 2b9c6711-4dd8-4e5c-9edc-dd106b8319b5 | test-snapshot                                            | active      |
    +--------------------------------------+----------------------------------------------------------+-------------+

Downloading snapshot
---------------------
1. Check the image ID of the snapshot

.. code-block:: bash

    $ openstack image list
    +--------------------------------------+----------------------------------------------------------+-------------+
    | ID                                   | Name                                                     | Status      |
    +--------------------------------------+----------------------------------------------------------+-------------+
    | 2b9c6711-4dd8-4e5c-9edc-dd106b8319b5 | test-snapshot                                            | active      |
    +--------------------------------------+----------------------------------------------------------+-------------+

2. Run

.. code-block:: bash

    openstack image save --file <file-name> <image-id>

.. code-block:: bash

    #example
    $ openstack image save --file snapshot.raw 0258526c-f523-4645-8a9d-f6980ad87864

Create new image from snapshot
------------------------------

Import snapshot to project
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

You must `Downloading snapshot`_ to local computer first.

1. Run

.. code-block:: bash

    openstack image create --container-format bare --disk-format qcow2 --file <path-to-image-file> <name>

.. code-block:: bash

    $ openstack image create --container-format bare --disk-format qcow2 --file snapshot.raw test-snapshot

Booting From Image
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Create a VM and selecting the image as the boot source

Example
"""""""

1. Find the ``image ID``

.. code-block:: bash

    $ openstack image list
    +--------------------------------------+----------------------------------------------------------+-------------+
    | ID                                   | Name                                                     | Status      |
    +--------------------------------------+----------------------------------------------------------+-------------+
    | 2b9c6711-4dd8-4e5c-9edc-dd106b8319b5 | test-snapshot                                            | active      |
    +--------------------------------------+----------------------------------------------------------+-------------+

2. Create the VM

.. code-block:: bash

    $ openstack server create --flavor c3.small --image 2b9c6711-4dd8-4e5c-9edc-dd106b8319b5 new-instance-from-snapshot


