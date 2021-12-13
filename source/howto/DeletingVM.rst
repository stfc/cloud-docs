==============================
Deleting Virtual Machine
==============================

Deleting entities is the only way to **release resources** and **free up quotas**.

.. warning::

    You should always refer to :ref:`Creating-Snapshot` as this process is **not reversible** and may result in **data loss**.

Web Interface
-------------

1. In Web Interface :guilabel:`Compute` → :guilabel:`Instances`, select the VM you wish to delete

.. image:: /assets/howtos/DeleteVM/Step1.png
    :align: center
    :alt:

2. Click :guilabel:`DELETE INSTANCES`

.. image:: /assets/howtos/DeleteVM/Step2.png
    :align: center
    :alt:

Command-line
-------------

See :ref:`Use-OpenStack-CLI` on how to set-up the command line client.

1. Find the instance ID with ``openstack server list``

.. code-block:: bash

    $ openstack server list
    +--------------------------------------+-----------------------+--------+-------------------------+---------------------------------------------------------+--------------+
    | ID                                   | Name                  | Status | Networks                | Image                                                   | Flavor       |
    +--------------------------------------+-----------------------+--------+-------------------------+---------------------------------------------------------+--------------+
    | c3b01eac-b5c7-4f57-8b09-f575f37cfe25 | delete-cli            | ACTIVE | Internal=172.16.113.107 | ubuntu-focal-20.04-nogui                                | c3.small     |
    +--------------------------------------+-----------------------+--------+-------------------------+---------------------------------------------------------+--------------+

2. Run

.. code-block:: bash

    openstack server delete <instance-id>

Example

.. code-block:: bash

    $ openstack server delete c3b01eac-b5c7-4f57-8b09-f575f37cfe25