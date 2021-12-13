=============================
Rebuild a VM with a new image
=============================

.. warning::

    Rebuilding Instances will result in loss of data. You should always refer to :ref:`Creating-Snapshot` to backup your data.

OpenStack has a useful rebuild function that allows you to rebuild an instance from a fresh image while maintaining the same fixed and floating IP addresses, amongst other metadata.

Web Interface
----------------------

1. Log-in to the STFC cloud (https://openstack.stfc.ac.uk/)
2. In the Web Interface, Go to :guilabel:`Compute` → :guilabel:`Instances`
3. Click the drop-down menu on the right-hand side (in :guilabel:`Actions` column) and select :guilabel:`REBUILD INSTANCE`

.. image:: /assets/howtos/RebuildVM/RebuildVM-1.png
    :align: center
    :alt:

4. Select the image from the drop-down menu in the pop-up. Click on the :guilabel:`REBUILD button`.

.. image:: /assets/howtos/RebuildVM/RebuildVM-2.png
    :align: center
    :alt:

5. The instance will be rebuilt with the new image

.. image:: /assets/howtos/RebuildVM/RebuildVM-3.png
    :align: center
    :alt:

Command-line
----------------------

.. note::
    
    See :ref:`Use-OpenStack-CLI` on how to set-up the command line client.

1. Get the ``Server ID`` (The VM to rebuild) and the ``Image ID``

.. code-block:: bash

    $ openstack image list
    +--------------------------------------+----------------------------------------------------------+-------------+
    | ID                                   | Name                                                     | Status      |
    +--------------------------------------+----------------------------------------------------------+-------------+
    | ae4a8e18-627c-484a-9909-7bd93d9e3e97 | ubuntu-focal-20.04-gui                                   | active      |
    +--------------------------------------+----------------------------------------------------------+-------------+
    $ openstack server list
    +--------------------------------------+--------------------------+--------+----------------------------------------+---------------------------------------------------------+--------------+
    | ID                                   | Name                     | Status | Networks                               | Image                                                   | Flavor       |
    +--------------------------------------+--------------------------+--------+----------------------------------------+---------------------------------------------------------+--------------+
    | b5acb398-76b4-48fe-9b9a-d480636fdfd9 | test-rebuild             | ACTIVE | Internal=172.16.101.195                | scientificlinux-7-nogui                                 | c3.small     |
    +--------------------------------------+--------------------------+--------+----------------------------------------+---------------------------------------------------------+--------------+

2. Run

.. code-block:: bash

    openstack server rebuild --image <image-id> <server-id>

Example

.. code-block:: bash

    $ openstack server rebuild --image ae4a8e18-627c-484a-9909-7bd93d9e3e97 b5acb398-76b4-48fe-9b9a-d480636fdfd9
    +-------------------+------------------------------------------------------------------+
    | Field             | Value                                                            |
    +-------------------+------------------------------------------------------------------+
    | OS-DCF:diskConfig | MANUAL                                                           |
    | accessIPv4        |                                                                  |
    | accessIPv6        |                                                                  |
    | addresses         | Internal=172.16.101.195                                          |
    | adminPass         | yoTq5HDrhH6a                                                     |
    | created           | 2021-12-03T10:31:27Z                                             |
    | flavor            | c3.small (026ace2c-5247-4bdc-8929-81d129cc69bf)                  |
    | hostId            | ff3c5830640867370bd8fb9228356baee63ff24e5baa381e41798dc9         |
    | id                | b5acb398-76b4-48fe-9b9a-d480636fdfd9                             |
    | image             | ubuntu-focal-20.04-gui (ae4a8e18-627c-484a-9909-7bd93d9e3e97)    |
    | name              | test-rebuild                                                     |
    | progress          | 0                                                                |
    | project_id        | 80ab2bd11e5f46bf96bf47658d07499d                                 |
    | properties        |                                                                  |
    | status            | REBUILD                                                          |
    | updated           | 2021-12-03T10:57:41Z                                             |
    | user_id           | 3ae4ecf4b9e0e66260b7aaebc2cc98aac3c95221e42f1cb49113ed751d8b9f2c |
    +-------------------+------------------------------------------------------------------+

