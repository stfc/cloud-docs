==============================
Security Group
==============================
A **security group** acts as a virtual **firewall** for servers and other resources on a network with a set of **security group rules** that dictates the network access for those resources.

.. contents:: Table of Contents

Create Security Group
--------------------------

Web Interface
^^^^^^^^^^^^^^^^^^^^^
1. Log-in to the STFC cloud (https://openstack.stfc.ac.uk/)
2. In the Web Interface, Go to :guilabel:`Network` → :guilabel:`Security Groups`
3. Click :guilabel:`CREATE SECURITY GROUP`

.. image:: /assets/howtos/SecGroup/Create-1.png
    :align: center
    :alt:

4. Give it a name and optionally a description and click :guilabel:`CREATE SECURITY GROUP`

.. image:: /assets/howtos/SecGroup/Create-2.png
    :align: center
    :alt:

5. See Security Group Rule Management for how to edit the security group rules.

Command-Line
^^^^^^^^^^^^^^^

.. note::
    
    See :ref:`Use-OpenStack-CLI` on how to set-up the command line client.

1. Run

.. code-block:: bash

    openstack security group create [--description <description>] <name>

Example

.. code-block:: bash

    $ openstack security group create test-tutor
    +-----------------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
    | Field           | Value                                                                                                                                                                                              |
    +-----------------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
    | created_at      | 2021-12-06T12:29:10Z                                                                                                                                                                               |
    | description     | test-tutor                                                                                                                                                                                         |
    | id              | e9c8ffad-2b5c-4a47-9887-9dc0ebffa8b5                                                                                                                                                               |
    | location        | Munch({'cloud': '', 'region_name': 'RegionOne', 'zone': None, 'project': Munch({'id': '80ab2bd11e5f46bf96bf47658d07499d', 'domain_id': '38372510d9bb4ac7916178b062d387de', 'domain_name': None})}) |
    | name            | test-tutor                                                                                                                                                                                         |
    | project_id      | 80ab2bd11e5f46bf96bf47658d07499d                                                                                                                                                                   |
    | revision_number | 1                                                                                                                                                                                                  |
    | rules           | created_at='2021-12-06T12:29:10Z', direction='egress', ethertype='IPv4', id='d23c7234-8f83-4307-84df-519aad87aa8c', updated_at='2021-12-06T12:29:10Z'                                              |
    |                 | created_at='2021-12-06T12:29:10Z', direction='egress', ethertype='IPv6', id='fc47d981-2368-403b-82bd-9ce6cdd9aa0d', updated_at='2021-12-06T12:29:10Z'                                              |
    | stateful        | None                                                                                                                                                                                               |
    | tags            | []                                                                                                                                                                                                 |
    | updated_at      | 2021-12-06T12:29:10Z                                                                                                                                                                               |
    +-----------------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+

Delete Security Group
--------------------------

Web Interface
^^^^^^^^^^^^^^^^^^^^^

1. Log-in to the STFC cloud (https://openstack.stfc.ac.uk/)
2. In the Web Interface, Go to :guilabel:`Network` → :guilabel:`Security Groups`
3. Select the security groups that you wish to delete and click :guilabel:`DELETE SECURITY GROUPS`

.. image:: /assets/howtos/SecGroup/Delete-1.png
    :align: center
    :alt:

4. Confirm by clicking :guilabel:`DELETE SECURITY GROUPS`

.. image:: /assets/howtos/SecGroup/Delete-2.png
    :align: center
    :alt:

Command-Line
^^^^^^^^^^^^^^^

.. note::
    
    See :ref:`Use-OpenStack-CLI` on how to set-up the command line client.

1. Find the security group ID with ``openstack server list``

.. code-block:: bash

    $ openstack security group list
    +--------------------------------------+---------------------+----------------------------------------------+----------------------------------+------+
    | ID                                   | Name                | Description                                  | Project                          | Tags |
    +--------------------------------------+---------------------+----------------------------------------------+----------------------------------+------+
    | d20fd393-8193-4068-a181-cf4861f112f7 | 1-group-to-delete   |                                              | 80ab2bd11e5f46bf96bf47658d07499d | []   |
    +--------------------------------------+---------------------+----------------------------------------------+----------------------------------+------+

2. Run

.. code-block:: bash

    openstack security group delete <security-group-id>

Example

.. code-block:: bash

    $ openstack security group delete d20fd393-8193-4068-a181-cf4861f112f7


Security Group Rule Management
------------------------------------
Security Group Rules define which traffic is allowed to instances assigned to the security group. A security group rule consists of three main parts:

* **Rule**: You can specify the desired rule template or use custom rules, the options are Custom TCP Rule, Custom UDP Rule, or Custom ICMP Rule.
* **Open Port/Port Range**: For TCP and UDP rules you may choose to open either a single port or a range of ports. Selecting the "Port Range" option will provide you with space to provide both the starting and ending ports for the range. For ICMP rules you instead specify an ICMP type and code in the spaces provided.
* **Remote**: You must specify the source of the traffic to be allowed via this rule. You may do so either in the form of an:

    * IP address block (CIDR)
    * Source group (Security Group) Selecting a security group as the source will allow any other instance in that security group access to any other instance via this rule.

Add Security Group Rule
^^^^^^^^^^^^^^^^^^^^^^^^^^^

Web Interface
"""""""""""""""""""

1. Log-in to the STFC cloud (https://openstack.stfc.ac.uk/)
2. In the Web Interface, Go to :guilabel:`Network` → :guilabel:`Security Groups`
3. Click :guilabel:`MANAGE RULES`

.. image:: /assets/howtos/SecGroup/Rule-Add-1.png
    :align: center
    :alt:

4. Click :guilabel:`ADD RULE`

.. image:: /assets/howtos/SecGroup/Rule-Add-2.png
    :align: center
    :alt:

5. Input the following:

* Rule: TCP, UDP or ICMP
* Direction

    * Ingress
    * Egress 

* Open Port:

    * Port
    * Port Range

* Port

    * the port number
    * You will see From port  and To port filed if you selected PORT RANGE 

* Remote (Source IP)

    * CIDR

        * Input the IP range

    * SECURITY GROUP

        * Select the Security group

.. image:: /assets/howtos/SecGroup/Rule-Add-3.png
    :align: center
    :alt:

6: Click :guilabel:`ADD`

Command-Line
"""""""""""""""""
1. Check existing security group using ``openstack security group list``

.. code-block:: bash

    $ openstack security group list
    +--------------------------------------+-------------------------+----------------------------------------------+----------------------------------+------+
    | ID                                   | Name                    | Description                                  | Project                          | Tags |
    +--------------------------------------+-------------------------+----------------------------------------------+----------------------------------+------+
    | c7e90937-af11-484c-8a21-90bea10e1407 | 1-group-rule-management |                                              | 80ab2bd11e5f46bf96bf47658d07499d | []   |
    +--------------------------------------+-------------------------+----------------------------------------------+----------------------------------+------+

.. code-block:: bash

    $ openstack security group rule list 1-group-rule-management
    +--------------------------------------+-------------+-----------+-----------+------------+-----------+-----------------------+----------------------+
    | ID                                   | IP Protocol | Ethertype | IP Range  | Port Range | Direction | Remote Security Group | Remote Address Group |
    +--------------------------------------+-------------+-----------+-----------+------------+-----------+-----------------------+----------------------+
    | 33cd09ee-0e9b-4ce3-b028-7e24f9604431 | None        | IPv6      | ::/0      |            | egress    | None                  | None                 |
    | 7969c6ae-dec0-4071-935e-6a81e7d8ec5c | None        | IPv4      | 0.0.0.0/0 |            | egress    | None                  | None                 |
    +--------------------------------------+-------------+-----------+-----------+------------+-----------+-----------------------+----------------------+

3. Create security group rule

.. list-table:: Title
   :widths: 25 25 50
   :header-rows: 1

   * - Argument
     - Description
     - Example
   * - <security-group-name>
     - Name of the security group
     - ``1-group-rule-management``
   * - <rule>
     - The protocol: TCP, UDP or ICMP
     - ``tcp``
   * - <port-range>
     - The range of ports to apply the rule: ``from_port:to_port``
     - ``89:90``
   * - <ip-range>
     - The source IP range for the rule
     - ``0.0.0.0/0``
   * - <source-security-group>
     - Name of the source security group to allow access
     - ``9200-Elastic-Search``
   * - [--ingress | --egress]
     - Ingress rule or egress rule (default is ``--ingress``)
     - ``--egress``    

3.a Security group rule based on traffic source IP

.. code-block:: bash
    
    openstack security group rule create <security-group-name> --protocol <rule> --dst-port <port-range> --remote-ip <ip-range> [--ingress | --egress]

Example

.. code-block:: bash

    $ openstack security group rule create 1-group-rule-management --protocol tcp --dst-port 89:90 --remote-ip 0.0.0.0/0
    $ openstack security group rule list 1-group-rule-management
    +--------------------------------------+-------------+-----------+-----------+------------+-----------+-----------------------+----------------------+
    | ID                                   | IP Protocol | Ethertype | IP Range  | Port Range | Direction | Remote Security Group | Remote Address Group |
    +--------------------------------------+-------------+-----------+-----------+------------+-----------+-----------------------+----------------------+
    | 33cd09ee-0e9b-4ce3-b028-7e24f9604431 | None        | IPv6      | ::/0      |            | egress    | None                  | None                 |
    | 7969c6ae-dec0-4071-935e-6a81e7d8ec5c | None        | IPv4      | 0.0.0.0/0 |            | egress    | None                  | None                 |
    | 7f4e2c68-a369-4be8-8491-561cccffc90c | tcp         | IPv4      | 0.0.0.0/0 | 89:90      | ingress   | None                  | None                 |
    +--------------------------------------+-------------+-----------+-----------+------------+-----------+-----------------------+----------------------+

3.b Security group rule based on source security group

.. code-block:: bash

    openstack security group rule create <security-group-name> --protocol <rule> --dst-port <port-range> --remote-group <source-security-group> [--ingress | --egress]

Example

.. code-block:: bash

    $ openstack security group rule create 1-group-rule-management --protocol tcp --dst-port 1021:1057 --remote-group 9200-Elastic-Search --egress
    $ openstack security group rule list 1-group-rule-management
    +--------------------------------------+-------------+-----------+-----------+------------+-----------+--------------------------------------+----------------------+
    | ID                                   | IP Protocol | Ethertype | IP Range  | Port Range | Direction | Remote Security Group                | Remote Address Group |
    +--------------------------------------+-------------+-----------+-----------+------------+-----------+--------------------------------------+----------------------+
    | 33cd09ee-0e9b-4ce3-b028-7e24f9604431 | None        | IPv6      | ::/0      |            | egress    | None                                 | None                 |
    | 7969c6ae-dec0-4071-935e-6a81e7d8ec5c | None        | IPv4      | 0.0.0.0/0 |            | egress    | None                                 | None                 |
    | f6bc9897-cc2c-40d8-aee1-3efe5107c394 | tcp         | IPv4      | 0.0.0.0/0 | 1021:1057  | egress    | 6f559f5d-9e92-4603-8226-de37cb39dcd8 | None                 |
    +--------------------------------------+-------------+-----------+-----------+------------+-----------+--------------------------------------+----------------------+

Delete Security Group Rule
^^^^^^^^^^^^^^^^^^^^^^^^^^^

Web Interface
"""""""""""""""
1. Log-in to the STFC cloud (https://openstack.stfc.ac.uk/)
2. In the Web Interface, Go to :guilabel:`Network` → :guilabel:`Security Groups`
3. Click :guilabel:`MANAGE RULES`

.. image:: /assets/howtos/SecGroup/Rule-Delete-1.png
    :align: center
    :alt:

4. Select the rules you want to delete and click :guilabel:`DELETE RULE`

.. image:: /assets/howtos/SecGroup/Rule-Delete-2.png
    :align: center
    :alt:

5. Click :guilabel:`DELETE RULE` to confirm delete

.. image:: /assets/howtos/SecGroup/Rule-Delete-3.png
    :align: center
    :alt:

Command-line
""""""""""""""""""
1. Get the security group name using ``openstack security group list``

.. code-block:: bash

    $ openstack security group list
    +--------------------------------------+-------------------------+----------------------------------------------+----------------------------------+------+
    | ID                                   | Name                    | Description                                  | Project                          | Tags |
    +--------------------------------------+-------------------------+----------------------------------------------+----------------------------------+------+
    | c7e90937-af11-484c-8a21-90bea10e1407 | 1-group-rule-management |                                              | 80ab2bd11e5f46bf96bf47658d07499d | []   |
    +--------------------------------------+-------------------------+----------------------------------------------+----------------------------------+------+

2. Get the ID of the rule with ``openstack security group rule list <security-group-name>``

.. code-block:: bash

    $ openstack security group rule list 1-group-rule-management
    +--------------------------------------+-------------+-----------+-----------+------------+-----------+-----------------------+----------------------+
    | ID                                   | IP Protocol | Ethertype | IP Range  | Port Range | Direction | Remote Security Group | Remote Address Group |
    +--------------------------------------+-------------+-----------+-----------+------------+-----------+-----------------------+----------------------+
    | 33cd09ee-0e9b-4ce3-b028-7e24f9604431 | None        | IPv6      | ::/0      |            | egress    | None                  | None                 |
    | 7969c6ae-dec0-4071-935e-6a81e7d8ec5c | None        | IPv4      | 0.0.0.0/0 |            | egress    | None                  | None                 |
    | 7f4e2c68-a369-4be8-8491-561cccffc90c | tcp         | IPv4      | 0.0.0.0/0 | 89:90      | ingress   | None                  | None                 |
    +--------------------------------------+-------------+-----------+-----------+------------+-----------+-----------------------+----------------------+

3. Run

.. code-block:: bash

    openstack security group rule delete <security-group-rule-id>

.. code-block:: bash

    $ openstack security group rule delete 7f4e2c68-a369-4be8-8491-561cccffc90c
    $ openstack security group rule list 1-group-rule-management
    +--------------------------------------+-------------+-----------+-----------+------------+-----------+-----------------------+----------------------+
    | ID                                   | IP Protocol | Ethertype | IP Range  | Port Range | Direction | Remote Security Group | Remote Address Group |
    +--------------------------------------+-------------+-----------+-----------+------------+-----------+-----------------------+----------------------+
    | 33cd09ee-0e9b-4ce3-b028-7e24f9604431 | None        | IPv6      | ::/0      |            | egress    | None                  | None                 |
    | 7969c6ae-dec0-4071-935e-6a81e7d8ec5c | None        | IPv4      | 0.0.0.0/0 |            | egress    | None                  | None                 |
    +--------------------------------------+-------------+-----------+-----------+------------+-----------+-----------------------+----------------------+



