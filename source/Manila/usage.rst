Manila: Shared File Systems as a Service 
###########################################

.. warning::

    OpenStack Manila is currently in technical preview on STFC Cloud. If you have any feedback or suggestions, please sent it to cloud-support@stfc.ac.uk

OpenStack Manila provides services for managing shared file systems and is the third storage OpenStack component alongside Cinder and Swift. 
Manila provides a way to provision remote, sharable file systems (shares) which can be accessed by multiple instances simultaneously and file systems can have access rules assigned to control read-write and read-only access to the file system.
On STFC Cloud, Manila provides a way to create **Ceph file sytems** (CephFS).



Manila Commands
-----------------
In OpenStack Train, the Manila CLI is mainly used for handling shares. There are some limited commands in the OpenStack CLI for shares.

.. note::

    To use Manila commands in OpenStack Train, the compatible version of the python package ``python-manilaclient`` needs to be installed. For OpenStack Train, this is `v2.0.0`. 
    This can be installed using ``pip install python-manilaclient=2.0.0``. 


OpenStack CLI 
~~~~~~~~~~~~~~~

In the OpenStack CLI the following commands are available:

.. code-block:: bash 

    openstack share --help

    share create # create a new share 
    share delete # delete a share 
    share list # list all shares
    share show # view details about a share


Manila CLI 
~~~~~~~~~~~~
Manila provides a CLI for not only for managing shares, but also for viewing the share types available, managing access rules for a share, and viewing the quota for share in your project.
Commands are run using ``manila <sub-command>``. The next section will demonstrate how to do some of the Manila commands. 

Usage
---------------

View Share Quota 
~~~~~~~~~~~~~~~~~~~

You can view either the overall quota using ``manila quota-show``, or the quota and the total quota used with ``manila absolute-limits``.

.. code-block:: bash 

    $ manila quota-show
    +-----------------------+----------------------------------+
    | Property              | Value                            |
    +-----------------------+----------------------------------+
    | gigabytes             | 250                              |
    | id                    | PROJECT_ID                       |
    | share_group_snapshots | 0                                |
    | share_groups          | 5                                |
    | share_networks        | 0                                |
    | shares                | 10                               |
    | snapshot_gigabytes    | 0                                |
    | snapshots             | 0                                |
    +-----------------------+----------------------------------+

.. code-block:: bash 

    $ manila absolute-limits
    +----------------------------+-------+
    | Name                       | Value |
    +----------------------------+-------+
    | maxTotalShareGigabytes     | 250   |
    | maxTotalShareNetworks      | 0     |
    | maxTotalShareSnapshots     | 0     |
    | maxTotalShares             | 10    |
    | maxTotalSnapshotGigabytes  | 250   |
    | totalShareGigabytesUsed    | 3     |
    | totalShareNetworksUsed     | 0     |
    | totalShareSnapshotsUsed    | 0     |
    | totalSharesUsed            | 3     |
    | totalSnapshotGigabytesUsed | 0     |
    +----------------------------+-------+

List Share Types Available 
~~~~~~~~~~~~~~~~~~~~~~~~~~~
Share types available can be listed using ``manila share-type list`` or ``openstack share list``. This will list the share types currently available and supported. 

.. code-block:: bash 

    $ openstack share list
    +--------------------------------------+-----------------------+------+-------------+-----------+-----------+-----------------+------+-------------------+
    | ID                                   | Name                  | Size | Share Proto | Status    | Is Public | Share Type Name | Host | Availability Zone |
    +--------------------------------------+-----------------------+------+-------------+-----------+-----------+-----------------+------+-------------------+
    | a6b215a6-00c5-46a5-b1db-d86559097896 | test_share            |    1 | CEPHFS      | available | False     | cephfs          |      | None              |
    | 7a1beb23-8dee-4709-9bcd-c947ae006653 | updated_demo_share    |    1 | CEPHFS      | available | False     | cephfs          |      | nova              |
    +--------------------------------------+-----------------------+------+-------------+-----------+-----------+-----------------+------+-------------------+

Create a Share 
~~~~~~~~~~~~~~~~~
Shares can be created by users using the manila create command. The required arguments are the share protocol and the size of the share in GiB. 

.. code-block:: bash 

   $ manila create --help
    usage: manila create [--snapshot-id <snapshot-id>] [--name <name>] [--metadata [<key=value> [<key=value> ...]]] [--share-network <network-info>] [--description <description>] [--share-type <share-type>]
                        [--public] [--availability-zone <availability-zone>] [--share-group <share-group>]
                        <share_protocol> <size>

    Creates a new share (NFS, CIFS, CephFS, GlusterFS, HDFS or MAPRFS).

    Positional arguments:
    <share_protocol>      Share protocol (NFS, CIFS, CephFS, GlusterFS, HDFS or MAPRFS).
    <size>                Share size in GiB.

    Optional arguments:
    --snapshot-id <snapshot-id>, --snapshot_id <snapshot-id>
                            Optional snapshot ID to create the share from. (Default=None)
    --name <name>         Optional share name. (Default=None)
    --metadata [<key=value> [<key=value> ...]]
                            Metadata key=value pairs (Optional, Default=None).
    --share-network <network-info>, --share_network <network-info>
                            Optional network info ID or name.
    --description <description>
                            Optional share description. (Default=None)
    --share-type <share-type>, --share_type <share-type>, --volume-type <share-type>, --volume_type <share-type>
                            Optional share type. Use of optional volume type is deprecated. (Default=None)
    --public              Level of visibility for share. Defines whether other tenants are able to see it or not. (Default=False)
    --availability-zone <availability-zone>, --availability_zone <availability-zone>, --az <availability-zone>
                            Availability zone in which share should be created.
    --share-group <share-group>, --share_group <share-group>, --group <share-group>
                            Optional share group name or ID in which to create the share (Experimental, Default=None). 

So if we want to create a new CephFS share of size 1GiB, we can use the following command:

.. code-block:: bash 

    $ manila create --name demo_share --description "Demo creating a share" --share-type SHARE_TYPE_ID CephFS 1
    +---------------------------------------+------------------------------------------------------------------+
    | Property                              | Value                                                            |
    +---------------------------------------+------------------------------------------------------------------+
    | status                                | creating                                                         |
    | share_type_name                       | cephfs                                                           |
    | description                           | Demo creating a share                                            |
    | availability_zone                     | None                                                             |
    | share_network_id                      | None                                                             |
    | share_group_id                        | None                                                             |
    | revert_to_snapshot_support            | False                                                            |
    | access_rules_status                   | active                                                           |
    | snapshot_id                           | None                                                             |
    | create_share_from_snapshot_support    | False                                                            |
    | is_public                             | False                                                            |
    | task_state                            | None                                                             |
    | snapshot_support                      | False                                                            |
    | id                                    | 7a1beb23-8dee-4709-9bcd-c947ae006653                             |
    | size                                  | 1                                                                |
    | source_share_group_snapshot_member_id | None                                                             |
    | user_id                               | USER_ID                                                          |
    | name                                  | demo_share                                                       |
    | share_type                            | SHARE_TYPE_ID                                                    |
    | has_replicas                          | False                                                            |
    | replication_type                      | None                                                             |
    | created_at                            | 2022-10-19T15:03:28.000000                                       |
    | share_proto                           | CEPHFS                                                           |
    | mount_snapshot_support                | False                                                            |
    | project_id                            | PROJECT_ID                                                       |
    | metadata                              | {}                                                               |
    +---------------------------------------+------------------------------------------------------------------+


Update a Share 
~~~~~~~~~~~~~~~~~
Once a share has been created in Manila, there are only three properties which can be updated:
- Name of the share.
- The description for the share.
- Change the visibility of the share to public or private.


.. code-block:: bash

    $ manila update --help
    usage: manila update [--name <name>] [--description <description>] [--is-public <is_public>] <share>

    Rename a share.

    Positional arguments:
      <share>               Name or ID of the share to rename.

    Optional arguments:
      --name <name>         New name for the share.
      --description <description>
                            Optional share description. (Default=None)
      --is-public <is_public>, --is_public <is_public>
                            Public share is visible for all tenants.

For example, we can update the name of a share from ``demo_share`` to ``updated_demo_share`` in the following way:

.. code-block:: bash 

    $ manila update --name updated_demo_share demo_share

Then we can see the updated share in the list of shares in the current project:

.. code-block:: bash 

    $ manila list
    +--------------------------------------+-----------------------+------+-------------+-----------+-----------+-----------------+------+-------------------+
    | ID                                   | Name                  | Size | Share Proto | Status    | Is Public | Share Type Name | Host | Availability Zone |
    +--------------------------------------+-----------------------+------+-------------+-----------+-----------+-----------------+------+-------------------+
    | 7a1beb23-8dee-4709-9bcd-c947ae006653 | updated_demo_share    | 1    | CEPHFS      | available | False     | cephfs          |      | nova              |
    +--------------------------------------+-----------------------+------+-------------+-----------+-----------+-----------------+------+-------------------+


Extend a Share 
~~~~~~~~~~~~~~~~~
The size of a share can be increased using the ``manila extend`` command.

.. code-block:: bash

    $ manila extend --help
    usage: manila extend <share> <new_size>

    Increases the size of an existing share.

    Positional arguments:
      <share>     Name or ID of share to extend.
      <new_size>  New size of share, in GiBs.

For example, if we want to extend a demo share from 1GiBs to 2GiBs, we can do the following:

.. code-block:: bash 

    $ manila extend updated_demo_share 2

Viewing the details of the share we can see that the size of the share has been updated.

.. code-block:: bash
    
    $ manila show updated_demo_share
    +---------------------------------------+-----------------------------------------------------------------------------------------------------------------------------+
    | Property                              | Value                                                                                                                       |
    +---------------------------------------+-----------------------------------------------------------------------------------------------------------------------------+
    | status                                | available                                                                                                                   |
    | share_type_name                       | cephfs                                                                                                                      |
    | description                           | Demo creating a share                                                                                                       |
    | availability_zone                     | nova                                                                                                                        |
    | share_network_id                      | None                                                                                                                        |
    | share_group_id                        | None                                                                                                                        |
    | revert_to_snapshot_support            | False                                                                                                                       |
    | access_rules_status                   | active                                                                                                                      |
    | snapshot_id                           | None                                                                                                                        |
    | create_share_from_snapshot_support    | False                                                                                                                       |
    | is_public                             | False                                                                                                                       |
    | task_state                            | None                                                                                                                        |
    | snapshot_support                      | False                                                                                                                       |
    | id                                    | 7a1beb23-8dee-4709-9bcd-c947ae006653                                                                                        |
    | size                                  | 2                                                                                                                           |
    | source_share_group_snapshot_member_id | None                                                                                                                        |
    | user_id                               | USER_ID                                                                                                                     |
    | name                                  | updated_demo_share                                                                                                          |
    | share_type                            | SHARE_TYPE                                                                                                                  |
    | has_replicas                          | False                                                                                                                       |
    | replication_type                      | None                                                                                                                        |
    | created_at                            | 2022-10-19T15:03:28.000000                                                                                                  |
    | share_proto                           | CEPHFS                                                                                                                      |
    | mount_snapshot_support                | False                                                                                                                       |
    | project_id                            | PROJECT_ID                                                                                                                  |
    | metadata                              | {}                                                                                                                          |
    | export_locations                      |                                                                                                                             |
    |                                       | path = EXPORT_PATH                                                                                                          |
    |                                       | id = EXPORT_LOCATIONS_ID                                                                                                    |
    |                                       | preferred = False                                                                                                           |
    +---------------------------------------+-----------------------------------------------------------------------------------------------------------------------------+


Shrink a Share 
~~~~~~~~~~~~~~~~

.. warning:: 

    This can only be done through the command line only.

The size of a share can be reduced using the ``manila shrink`` command.

.. code-block:: bash

    $ manila shrink --help
    usage: manila shrink <share> <new_size>

    Decreases the size of an existing share.

    Positional arguments:
      <share>     Name or ID of share to shrink.
      <new_size>  New size of share, in GiBs.

Using the example in the previous section, we can reduce the size of a share from 2GiB to 1Gib using:

.. code-block:: bash 

    $ manila shrink updated_demo_share 1

We can see using manila show ``updated_demo_share`` that the size of the share has been updated:

.. code-block:: bash

    $ manila show updated_demo_share
    +---------------------------------------+-----------------------------------------------------------------------------------------------------------------------------+
    | Property                              | Value                                                                                                                       |
    +---------------------------------------+-----------------------------------------------------------------------------------------------------------------------------+
    | status                                | available                                                                                                                   |
    | share_type_name                       | cephfs                                                                                                                      |
    | description                           | Demo creating a share                                                                                                       |
    | availability_zone                     | nova                                                                                                                        |
    | share_network_id                      | None                                                                                                                        |
    | share_group_id                        | None                                                                                                                        |
    | revert_to_snapshot_support            | False                                                                                                                       |
    | access_rules_status                   | active                                                                                                                      |
    | snapshot_id                           | None                                                                                                                        |
    | create_share_from_snapshot_support    | False                                                                                                                       |
    | is_public                             | False                                                                                                                       |
    | task_state                            | None                                                                                                                        |
    | snapshot_support                      | False                                                                                                                       |
    | id                                    | 7a1beb23-8dee-4709-9bcd-c947ae006653                                                                                        |
    | size                                  | 1                                                                                                                           |
    | source_share_group_snapshot_member_id | None                                                                                                                        |
    | user_id                               | USER_ID                                                                                                                     |
    | name                                  | updated_demo_share                                                                                                          |
    | share_type                            | SHARE_TYPE                                                                                                                  |
    | has_replicas                          | False                                                                                                                       |
    | replication_type                      | None                                                                                                                        |
    | created_at                            | 2022-10-19T15:03:28.000000                                                                                                  |
    | share_proto                           | CEPHFS                                                                                                                      |
    | mount_snapshot_support                | False                                                                                                                       |
    | project_id                            | PROJECT_ID                                                                                                                  |
    | metadata                              | {}                                                                                                                          |
    | export_locations                      |                                                                                                                             |
    |                                       | path = EXPORT_PATH                                                                                                          |
    |                                       | id = EXPORT_LOCATIONS_ID                                                                                                    |
    |                                       | preferred = False                                                                                                           |
    +---------------------------------------+-----------------------------------------------------------------------------------------------------------------------------+


Add Access Rule through CLI 
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Access rules can be created for a share through the Web UI or on the command line. 

.. warning::

    Only cephx access rules can be used for shares as only CephFS shares are currently supported. Any other type of access rule created will go into error state.


To create a new access rule for a share, e.g. a share named ``demo_share``, we can use the ``manila access-allow`` command:

.. code-block:: bash 

    $ manila access-allow --help
    usage: manila access-allow [--access-level <access_level>] [--metadata [<key=value> [<key=value> ...]]] <share> <access_type> <access_to>

    Allow access to a given share.

    Positional arguments:
      <share>               Name or ID of the NAS share to modify.
      <access_type>         Access rule type (only "ip", "user"(user or group), "cert" or "cephx" are supported).
      <access_to>           Value that defines access.

    Optional arguments:
      --access-level <access_level>, --access_level <access_level>
                            Share access level ("rw" and "ro" access levels are supported). Defaults to rw.
      --metadata [<key=value> [<key=value> ...]]
                            Space Separated list of key=value pairs of metadata items. OPTIONAL: Default=None.

    $ manila access-allow demo_share cephx alice

Then we can view the access rules for the share using:

.. code-block:: bash 

    $ manila access-list demo_share 
    +--------------------------------------+-------------+----------------+--------------+--------+------------------------------------------+----------------------------+----------------------------+
    | id                                   | access_type | access_to      | access_level | state  | access_key                               | created_at                 | updated_at                 |
    +--------------------------------------+-------------+----------------+--------------+--------+------------------------------------------+----------------------------+----------------------------+
    | 56907a0c-024a-465e-8ebc-5a0b085ac87b | cephx       | alice          | rw           | active | ACCESS_KEY                               | 2022-10-14T14:55:59.000000 | 2022-10-14T14:55:59.000000 |
    +--------------------------------------+-------------+----------------+--------------+--------+------------------------------------------+----------------------------+----------------------------+



Remove Access Rule through CLI 
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Access rules can be removed from a share using ``manila access-deny``:

.. code-block:: bash 

    $ manila access-deny --help
    usage: manila access-deny <share> <id>

    Deny access to a share.

    Positional arguments:
      <share>  Name or ID of the NAS share to modify.
      <id>     ID of the access rule to be deleted.


Delete a Share 
~~~~~~~~~~~~~~~~~

A share can be deleted by using the manila delete command:

.. code-block:: bash

    $ manila delete --help
    usage: manila delete [--share-group <share-group>] <share> [<share> ...]

    Remove one or more shares.

    Positional arguments:
      <share>               Name or ID of the share(s).

    Optional arguments:
      --share-group <share-group>, --share_group <share-group>, --group <share-group>
                            Optional share group name or ID which contains the share (Experimental, Default=None).


How to access a CephFS Share 
-------------------------------

There are two methods that can be used to mount a share onto a VM:

- Kernel method 
  
- Ceph FUSE method 

The following examples assume that the following have been set up:

- VM (examples use an Ubuntu VM)
  
- CephFS Share 
  
- cephx share access rules 


Access a share using the Kernel Client
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The kernel client requires the ``ceph-common`` package to be installed. To access the filesystem on a share, we can use 
the ``mount`` command of the form:

.. code-block:: bash 

    mount -t ceph {mon1 ip addr}:6789,{mon2 ip addr}:6789,{mon3 ip addr}:6789:/{mount-point} -o name={access-id},secret={access-key}


Access a share using the FUSE Client 
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

In order to use the FUSE client, make sure the version of the ``ceph-fuse`` package is at least ``octopus``. The version of the package can be found using:

.. code-block:: bash 

    ~$ ceph-fuse --version
    ceph version 15.2.16 (d46a73d6d0a67a79558054a3a5a72cb561724974) octopus (stable)


To use the FUSE client, we need to set up a ``keyring`` and a ``ceph.conf`` file. If we have named the cephx rule ``alice``, then 
the keyring file will need to be named ``alice.keyring`` and needs to contain the following:

.. code-block:: bash 
    
    [client.alice]
        key = CEPHX_KEY

Then for the ``ceph.conf`` file, we need to add the IP addresses for the monitors from the ceph cluster. 
The IP address for the ceph monitors for the share can be found using ``manila show <share-id>`` under the `export locations` field.

.. code-block:: bash 

    [client]
        client quota = True
        mon host = MON_HOST_IP_ADDRESSES


Then the filesystem can be mounted to a ``test`` directory using the ``alice`` cephx access rule:

.. code-block:: bash

    sudo ceph-fuse ~/mnt \
    --id=alice \
    --conf=./ceph.conf \
    --keyring=./alice.keyring \
    --client-mountpoint={mount-point}

Here the mount point is the path to the share on the ceph cluster. 
The share path can be found using ``manila show <share-id>`` under the ``export locations`` field.



References:
-------------

https://docs.openstack.org/manila/train/user/create-and-manage-shares.html

https://docs.openstack.org/manila/train/admin/shared-file-systems-crud-share.html#manage-access-to-share

https://docs.openstack.org/manila/train/admin/cephfs_driver.html