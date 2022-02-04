.. _Resize_VM:

============
Resize a VM
============

You can change the size of an instance by changing its flavor. 

.. warning::

    This is not reversible! Rebuilding Instances can result in loss of data. You should always refer to :ref:`Creating-Snapshot` to backup your data.

.. note::

    This rebuilds the instance and therefore results in a restart

Web Interface
----------------------
1. Log-in to the STFC cloud (https://openstack.stfc.ac.uk/)
2. In the Web Interface, Go to :guilabel:`Compute` → :guilabel:`Instances`
3. Shut down the Instance and create a snapshot to prevent data loss.
4. Click the drop-down menu on the right-hand side (in :guilabel:`Actions` column) and select :guilabel:`RESIZE  INSTANCE`

.. image:: /assets/howtos/ResizeVM/ResizeVM-1.png
    :align: center
    :alt:

5. Select the new flavor from the drop-down menu in the pop-up. Click on the :guilabel:`RESIZE` button.

.. image:: /assets/howtos/ResizeVM/ResizeVM-2.png
    :align: center
    :alt:

Command-line
----------------------

.. note::
    
    See :ref:`Use-OpenStack-CLI` on how to set-up the command line client.

1. Get the ``serverID`` (the VM to resize) and ``flavorID`` 

.. code-block:: bash

    $ openstack server list
    +--------------------------------------+--------------------------+---------+----------------------------------------+---------------------------------------------------------+--------------+
    | ID                                   | Name                     | Status  | Networks                               | Image                                                   | Flavor       |
    +--------------------------------------+--------------------------+---------+----------------------------------------+---------------------------------------------------------+--------------+
    | b5acb398-76b4-48fe-9b9a-d480636fdfd9 | test-rebuild             | SHUTOFF | Internal=172.16.101.195                | ubuntu-focal-20.04-gui                                  | c3.small     |
    +--------------------------------------+--------------------------+---------+----------------------------------------+---------------------------------------------------------+--------------+
    $ openstack flavor list
    +--------------------------------------+--------------+--------+------+-----------+-------+-----------+
    | ID                                   | Name         |    RAM | Disk | Ephemeral | VCPUs | Is Public |
    +--------------------------------------+--------------+--------+------+-----------+-------+-----------+
    | 6cf0813c-1ba2-4999-b7eb-34d71a2a4199 | c3.medium    |   8192 |   40 |         0 |     4 | True      |
    +--------------------------------------+--------------+--------+------+-----------+-------+-----------+

2. Run

.. code-block:: bash

    openstack server resize --flavor <falvor-id> <server-id>

Example

.. code-block:: bash

    $ openstack server resize --flavor 6cf0813c-1ba2-4999-b7eb-34d71a2a4199 b5acb398-76b4-48fe-9b9a-d480636fdfd9
