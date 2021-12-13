.. _Use-OpenStack-CLI:

==============================================================================================
Using OpenStack Command-line Interface
==============================================================================================

This tutorial is based on ``Ubuntu 20.04``. At the end of the tutorial, you will have a machine that can control your cloud project using OpenStack CLI.


Install client
--------------
1. Install ``python``, ``pip`` and ``openstackclient``

.. code-block:: bash

    sudo apt install python-dev python3-pip python3-openstackclient

Log-in with Openstack CLI
-------------------------
2. in :guilabel:`Project` → :guilabel:`API Access`

.. image:: /assets/howtos/UsingOpenStackCommand-lineInterface/Step2.png
    :align: center
    :alt:

3. Press :guilabel:`DOWNLOAD OPENSTACK RC FILE` drop-down
4. Press :guilabel:`Openstack RC FILE` to download the rc file.

.. image:: /assets/howtos/UsingOpenStackCommand-lineInterface/Step4.png
    :align: center
    :alt:

5. Run

.. code-block:: bash

    source <path-to-openstack-rc-file>

Verify connection
-------------------------
6. Run ``openstack server list`` and check if you can see the instances

.. code-block:: bash

    $ openstack server list
    +--------------------------------------+--------------------------+--------+----------------------------------------+---------------------------------------------------------+--------------+
    | ID                                   | Name                     | Status | Networks                               | Image                                                   | Flavor       |
    +--------------------------------------+--------------------------+--------+----------------------------------------+---------------------------------------------------------+--------------+
    | 6b2bedc4-9d8e-4bf3-be63-1dd49bc2e188 | test-resize-rebuild      | ACTIVE | Internal=172.16.102.207                | ubuntu-focal-20.04-gui                                  | c3.small     |
    +--------------------------------------+--------------------------+--------+----------------------------------------+---------------------------------------------------------+--------------+
