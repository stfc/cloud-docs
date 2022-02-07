======================================================================================================
Grow the disk of an Openstack VM by cloning the machine
======================================================================================================

.. contents:: Table of Contents

##########
Scenario
##########
In the STFC cloud, if you created a new VM in instances, the default setting in the :guilabel:`Sources` was to create the VM with a image. If you run out of disk space on the VM, you can usually :guilabel:`resize` it to a new Flavor, and you will have the same VM, but with more space.

.. note::

    See :ref:`Resize_VM` on how to resize a VM.

You can also create a new instance with :guilabel:`Volume` as :guilabel:`Sources`. However, with :guilabel:`Volumes`, the system ignores the disk sizing aspect of the :guilabel:`flavor`. This document describes how to create a clone of the original VM and increase the disk size, so that space is available on the new VM.

###########
Method
###########

WebUI
********************
.. note::

    Check your Quotas for our project: Volume sizes, number of volumes, and number of volume snapshots are all parameters for which your project has a quota for, and there is a fair chance that what you have allocated may not be enough. Check with a cloud admin first before proceeding if you are unsure.

1. Go to :guilabel:`Compute` → :guilabel:`Instances` in the LHS menu
2. Click the drop-down menu on the right-hand side (in :guilabel:`Actions` column) and select :guilabel:`SHUT OFF INSTANCE`

.. image:: /assets/howtos/GrowDisk/GrowDisk-1.png
    :align: center
    :alt:

3. Click on the instance name

.. image:: /assets/howtos/GrowDisk/GrowDisk-2.png
    :align: center
    :alt:

4. Check the :guilabel:`Volumes Attached` for the volume attached to `/dev/vda` and click on the volume name

.. image:: /assets/howtos/GrowDisk/GrowDisk-3.png
    :align: center
    :alt:

5. Open the drop-down menu on the top left corner and click :guilabel:`CREATE SNAPSHOT`

.. image:: /assets/howtos/GrowDisk/GrowDisk-4.png
    :align: center
    :alt:

6. Give it a name and click :guilabel:`CREATE VOLUME SNAPSHOT (FORCE)`

.. image:: /assets/howtos/GrowDisk/GrowDisk-5.png
    :align: center
    :alt:

.. note::

    Once the volume is attached to a VM, you cannot remove it: the only way to remove the VM host from the volume is if the instance is created, and when the VM was created, the setting for the volume to persist still exists.

7. Go to :guilabel:`Volume` → :guilabel:`Volumes` in the LHS menu, then click on :guilabel:`+ CREATE VOLUME`

.. image:: /assets/howtos/GrowDisk/GrowDisk-6.png
    :align: center
    :alt:

8. Give it a name an in :guilabel:`Volume Source` Select :guilabel:`SNAPSHOT`; In :guilabel:`Use snapshot as a source` select the snapshot you created and input the :guilabel:`Size (GiB)` you need. After that, Click :guilabel:`CREATE VOLUME`

.. image:: /assets/howtos/GrowDisk/GrowDisk-7.png
    :align: center
    :alt:

9. Refer to :ref:`VM_Create_WebUI` except in :guilabel:`Source` you should select :guilabel:`Select Boot Source` → :guilabel:`Volume` and select the volume you created from the snapshot

.. note::

    The flavor of the new VM does not affect the disk size – only the volume parameters does this !

.. image:: /assets/howtos/GrowDisk/GrowDisk-8.png
    :align: center
    :alt:

.. note::

    When creating a new instance from a volume, the volume may still have previous users created in the system and their home directories and they may have their details still in the ``/etc/sudoers.d/cloud`` file. However, you will not find that your :guilabel:`FEDID` has been added to the ``/etc/sudoers.d/cloud`` file.

Command-Line
********************
1. Get the ID for the ``Instance`` you want to grow with ``openstack server list``.

.. code-block:: bash

    $ openstack server list
    +--------------------------------------+--------------------------+---------+-------------------------+---------------------------------------------------------+--------------+
    | ID                                   | Name                     | Status  | Networks                | Image                                                   | Flavor       |
    +--------------------------------------+--------------------------+---------+-------------------------+---------------------------------------------------------+--------------+
    | 2f2c54ac-bae0-4f26-a4bb-7e9ff429f3b4 | volume launch 2          | ACTIVE  | Internal=172.16.101.212 |                                                         | c3.small     |
    +--------------------------------------+--------------------------+---------+-------------------------+---------------------------------------------------------+--------------+

2. Get the details of the VM with ``openstack server show <Instance-ID>`` Note down the ``ID`` of ``volumes_attached``.

.. code-block:: bash

    $ openstack server show 2f2c54ac-bae0-4f26-a4bb-7e9ff429f3b4
    +-----------------------------+------------------------------------------------------------------+
    | Field                       | Value                                                            |
    +-----------------------------+------------------------------------------------------------------+
    | OS-DCF:diskConfig           | AUTO                                                             |
    | OS-EXT-AZ:availability_zone | ceph                                                             |
    | OS-EXT-STS:power_state      | Running                                                          |
    | OS-EXT-STS:task_state       | None                                                             |
    | OS-EXT-STS:vm_state         | active                                                           |
    | OS-SRV-USG:launched_at      | 2022-02-04T08:24:54.000000                                       |
    | OS-SRV-USG:terminated_at    | None                                                             |
    | accessIPv4                  |                                                                  |
    | accessIPv6                  |                                                                  |
    | addresses                   | Internal=172.16.101.212                                          |
    | config_drive                | True                                                             |
    | created                     | 2022-02-04T08:24:31Z                                             |
    | flavor                      | c3.small (026ace2c-5247-4bdc-8929-81d129cc69bf)                  |
    | hostId                      | 206ef5cda6ec5d9c47a3fed0c7cf1980c98cf4b11e870195380699ca         |
    | id                          | 2f2c54ac-bae0-4f26-a4bb-7e9ff429f3b4                             |
    | image                       |                                                                  |
    | key_name                    | None                                                             |
    | name                        | volume launch 2                                                  |
    | progress                    | 0                                                                |
    | project_id                  | 80ab2bd11e5f46bf96bf47658d07499d                                 |
    | properties                  |                                                                  |
    | security_groups             | name='default'                                                   |
    | status                      | ACTIVE                                                           |
    | updated                     | 2022-02-04T08:24:54Z                                             |
    | user_id                     | 3ae4ecf4b9e0e66260b7aaebc2cc98aac3c95221e42f1cb49113ed751d8b9f2c |
    | volumes_attached            | id='d7f3c63e-4d01-4959-bde7-fe55032243fd'                        |
    +-----------------------------+------------------------------------------------------------------+

3. Shutdown the VM using ``openstack server stop <Instance-ID>``

.. code-block:: bash

    $ openstack server stop 2f2c54ac-bae0-4f26-a4bb-7e9ff429f3b4

4. Create a snapshot using ``openstack volume snapshot create --volume  --force <snapshot-name>``

.. code-block:: bash

    $ openstack volume snapshot create --volume d7f3c63e-4d01-4959-bde7-fe55032243fd --force test-cli-grow-snapshot
    +-------------+--------------------------------------+
    | Field       | Value                                |
    +-------------+--------------------------------------+
    | created_at  | 2022-02-04T13:44:49.884301           |
    | description | None                                 |
    | id          | 2e09549b-b02c-494f-a704-0a882d0ef96c |
    | name        | test-cli-grow-snapshot               |
    | properties  |                                      |
    | size        | 30                                   |
    | status      | creating                             |
    | updated_at  | None                                 |
    | volume_id   | d7f3c63e-4d01-4959-bde7-fe55032243fd |
    +-------------+--------------------------------------+

5. Create a volume using the snapshot ``openstack volume create --snapshot <snapshot-id> --size <size-in-GiB> <name>``

.. code-block:: bash    

    $ openstack volume create --snapshot 2e09549b-b02c-494f-a704-0a882d0ef96c --size 35 test-grow-cli-volume
    +---------------------+------------------------------------------------------------------+
    | Field               | Value                                                            |
    +---------------------+------------------------------------------------------------------+
    | attachments         | []                                                               |
    | availability_zone   | ceph                                                             |
    | bootable            | true                                                             |
    | consistencygroup_id | None                                                             |
    | created_at          | 2022-02-04T13:48:24.000000                                       |
    | description         | None                                                             |
    | encrypted           | False                                                            |
    | id                  | 37c8b414-f2c9-452c-885b-8cffc185c596                             |
    | multiattach         | False                                                            |
    | name                | test-grow-cli-volume                                             |
    | properties          |                                                                  |
    | replication_status  | None                                                             |
    | size                | 35                                                               |
    | snapshot_id         | 2e09549b-b02c-494f-a704-0a882d0ef96c                             |
    | source_volid        | None                                                             |
    | status              | creating                                                         |
    | type                | __DEFAULT__                                                      |
    | updated_at          | None                                                             |
    | user_id             | 3ae4ecf4b9e0e66260b7aaebc2cc98aac3c95221e42f1cb49113ed751d8b9f2c |
    +---------------------+------------------------------------------------------------------+

6. Create a VM using the volume ``openstack server create --volume <volume-id> --flavor <flavor> --network <network> <name>``

.. code-block:: bash    

    $ openstack server create --volume 37c8b414-f2c9-452c-885b-8cffc185c596 --flavor c3.small --network Internal test-cli-grow-vm
    +-----------------------------+------------------------------------------------------------------+
    | Field                       | Value                                                            |
    +-----------------------------+------------------------------------------------------------------+
    | OS-DCF:diskConfig           | MANUAL                                                           |
    | OS-EXT-AZ:availability_zone |                                                                  |
    | OS-EXT-STS:power_state      | NOSTATE                                                          |
    | OS-EXT-STS:task_state       | scheduling                                                       |
    | OS-EXT-STS:vm_state         | building                                                         |
    | OS-SRV-USG:launched_at      | None                                                             |
    | OS-SRV-USG:terminated_at    | None                                                             |
    | accessIPv4                  |                                                                  |
    | accessIPv6                  |                                                                  |
    | addresses                   |                                                                  |
    | adminPass                   | wXJMroU7AhVZ                                                     |
    | config_drive                |                                                                  |
    | created                     | 2022-02-04T13:50:31Z                                             |
    | flavor                      | c3.small (026ace2c-5247-4bdc-8929-81d129cc69bf)                  |
    | hostId                      |                                                                  |
    | id                          | f3d5f09a-74f1-42ea-8823-a45429aa0eb9                             |
    | image                       |                                                                  |
    | key_name                    | None                                                             |
    | name                        | test-cli-grow-vm                                                 |
    | progress                    | 0                                                                |
    | project_id                  | 80ab2bd11e5f46bf96bf47658d07499d                                 |
    | properties                  |                                                                  |
    | security_groups             | name='default'                                                   |
    | status                      | BUILD                                                            |
    | updated                     | 2022-02-04T13:50:31Z                                             |
    | user_id                     | 3ae4ecf4b9e0e66260b7aaebc2cc98aac3c95221e42f1cb49113ed751d8b9f2c |
    | volumes_attached            |                                                                  |
    +-----------------------------+------------------------------------------------------------------+

###################################################
Confirm the storage 
###################################################

You can confirm the size of the disk using ``lsblk``

.. code-block:: bash  

    $ lsblk
    NAME    MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
    loop0     7:0    0 61.9M  1 loop /snap/core20/1242
    loop1     7:1    0 55.5M  1 loop /snap/core18/2253
    loop2     7:2    0 73.1M  1 loop /snap/lxd/21902
    loop3     7:3    0 55.5M  1 loop /snap/core18/2284
    loop4     7:4    0 32.5M  1 loop /snap/snapd/13640
    loop5     7:5    0 43.4M  1 loop /snap/snapd/14549
    loop6     7:6    0 76.2M  1 loop /snap/lxd/22340
    loop7     7:7    0 61.9M  1 loop /snap/core20/1328
    sr0      11:0    1  470K  0 rom  /mnt/context
    vda     252:0    0   35G  0 disk
    ├─vda1  252:1    0 34.9G  0 part /
    ├─vda14 252:14   0    4M  0 part
    └─vda15 252:15   0  106M  0 part /boot/efi