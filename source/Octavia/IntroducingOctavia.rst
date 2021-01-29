Octavia
========

Octavia provides network load balancing for OpenStack and is the reference implementation LBaaS  (Load Balancer as a Service). Since the OpenStack Liberty release, Octavia has fully replaced and has become the reference implementation for Load Balancing as a Service (LBaaS v2).

    Neutron previously provided reference implementation for LBaaS, however it has since been deprecated and Octavia should be used for creating load balancers. OpenStack strongly advises that users should use Octavia to create Load balancers.

Load balancers can be created through the web UI or the OpenStack CLI.

Octavia works with the OpenStack Projects to provide a service for creating LBaaS pools:

- **Nova:** managing amphora lifecycle and creating compute resources.
- **Neutron:** For network connectivity
- **Keystone:** Authentication against Octavia API
- **Glance:** stores amphora VM image.
- **Oslo:** Communication between Octavia components. Octavia makes extensive use of **Taskflow**.


LBaaS Pools
-----------

LBaaS pools may consist of the following:

**Load Balancer:** Balances the traffic to server pools. In combination with the health monitor and listener, the load balancer can redistribute traffic to servers in the event of one server failing for example.

**Health Monitor:** Monitors the health of the pool and defines the method for checking the health of each member of the pool.

**Listener:** Represent a listening endpoint for the VIP. Here, the Listener listens for client traffic being directed to the load balanced pool.

**Pool:** A group of servers which are identified by their floating IP addresses. The size of the pool can be increased or decreased in response to traffic to the instances detected by the load balancer.

These can be created through the web UI, command line, or as part of a Heat Stack such as a LAMP stack.


Octavia CLI
------------

In order to use the commands provided from Octavia, make sure to you have the following python package installed:

.. code-block:: bash

  pip install python-octaviaclient


To test whether the octaviaclient has been installed successfully and we can access the Octavia commands, we can try to list load balancers in our current project:

.. code-block:: bash

  openstack loadbalancer list


This should return an empty line if there are no load balancers in your project, or a table containing details of load balancers similar to the table below:

.. code-block:: bash

  +--------------------------------------+---------------------------+----------------------------------+----------------+---------------------+----------+
  | id                                   | name                      | project_id                       | vip_address    | provisioning_status | provider |
  +--------------------------------------+---------------------------+----------------------------------+----------------+---------------------+----------+
  | LOADBALANCER_ID                      | LOADBALANCER 1            | PROJECT_ID                       | 192.168.132.93 | ACTIVE              | amphora  |
  | LOADBALANCER_ID                      | LOADBALANCER 2            | PROJECT_ID                       | 10.0.0.38      | ACTIVE              | amphora  |
  | LOADBALANCER_ID                      | LOADBALANCER 3            | PROJECT_ID                       | 10.0.0.254     | ACTIVE              | amphora  |
  +--------------------------------------+---------------------------+----------------------------------+----------------+---------------------+----------+



Commands
~~~~~~~~

Having installed ``python-octaviaclient``, we are now able to run commands for working with load balanced pools.

There are several commands which Octavia provides which we can use for working with loadbalancers, pools, listeners, and health monitors. Below is a list of some of the commands which can be used:

.. code-block:: bash

  #Commands are of the form:
  openstack loadbalancer <commands>

  #Load Balancers
  loadbalancer create # create a load balancer
  loadbalancer delete # delete a load balancer
  loadbalancer list # list load balancer
  loadbalancer show # view details for a load balancer
  loadbalancer failover # trigger a load balancer failover
  loadbalancer set # update a loadbalancer
  loadbalancer stats show # show current stats for the load balancer
  loadbalancer status show # show status of the load balancer
  loadbalancer unset #clear loadbalancer settings

  #Health Monitor
  loadbalancer healthmonitor create #create a health monitor
  loadbalancer healthmonitor delete #delete a health monitor
  loadbalancer healthmonitor list #list health monitors
  loadbalancer healthmonitor set #update health monitors
  loadbalancer healthmonitor show #view health monitor details
  loadbalancer healthmonitor unset #clear health monitor settings

  #Listener
  loadbalancer listener create #create listener
  loadbalancer listener delete #delete listener
  loadbalancer listener list #list listeners
  loadbalancer listener set #update listener
  loadbalancer listener show #view details of a listener
  loadbalancer listener stats show #show listener stats
  loadbalancer listener unset #clear settings for listener

  #Loadbalancer members
  loadbalancer member create #create a pool member
  loadbalancer member delete #remove pool member
  loadbalancer member list #list pool members
  loadbalancer member set #update a pool member
  loadbalancer member show #view details of a pool member
  loadbalancer member unset #clear settings for pool member

  #Loadbalanced pools
  loadbalancer pool create #create loadbalancer pool
  loadbalancer pool delete #delete pool
  loadbalancer pool list #list pools
  loadbalancer pool set #update pool
  loadbalancer pool show #view details of the pool
  loadbalancer pool unset #clear pool settings


Further commands can be seen using ``openstack loadbalancer --help``.

Creating LBaaS v2 Pools
-----------------------

LBaaS v2 pools can be created through the Web UI and CLI. Let's look at creating a simple LBaaS v2 pool.

Load Balancer
~~~~~~~~~~~~~~

Load balancers can be created using:

.. code-block:: bash

  openstack loadbalancer create [-h] [-f {json,shell,table,value,yaml}] [-c COLUMN] [--noindent] [--prefix PREFIX]
                                       [--max-width <integer>] [--fit-width] [--print-empty] [--name <name>]
                                       [--description <description>] [--vip-address <vip_address>] [--vip-port-id <vip_port_id>]
                                       [--vip-subnet-id <vip_subnet_id>] [--vip-network-id <vip_network_id>]
                                       [--vip-qos-policy-id <vip_qos_policy_id>] [--project <project>] [--provider <provider>]
                                       [--availability-zone <availability_zone>] [--enable | --disable] [--flavor <flavor>] [--wait]

  optional arguments:
    -h, --help            show this help message and exit
    --name <name>         New load balancer name.
    --description <description>
                          Set load balancer description.
    --vip-address <vip_address>
                          Set the VIP IP Address.
    --vip-qos-policy-id <vip_qos_policy_id>
                          Set QoS policy ID for VIP port. Unset with 'None'.
    --project <project>   Project for the load balancer (name or ID).
    --provider <provider>
                          Provider name for the load balancer.
    --availability-zone <availability_zone>
                          Availability zone for the load balancer.
    --enable              Enable load balancer (default).
    --disable             Disable load balancer.
    --flavor <flavor>     The name or ID of the flavor for the load balancer.
    --wait                Wait for action to complete



For example, we can create a load balancer that is on a private subnet.

.. code-block:: bash

  openstack loadbalancer create --name test-lb --vip-subnet-id <subnet-id>

  #This should return a table similar to:
  +---------------------+--------------------------------------+
  | Field               | Value                                |
  +---------------------+--------------------------------------+
  | admin_state_up      | True                                 |
  | availability_zone   |                                      |
  | created_at          | 2020-11-11T17:00:19                  |
  | description         |                                      |
  | flavor_id           | None                                 |
  | id                  | fe22e256-d409-4f91-867f-508d08566892 |
  | listeners           |                                      |
  | name                | test-lb                              |
  | operating_status    | OFFLINE                              |
  | pools               |                                      |
  | project_id          | PROJECT_ID                          |
  | provider            | amphora                              |
  | provisioning_status | PENDING_CREATE                       |
  | updated_at          | None                                 |
  | vip_address         | 192.168.132.125                      |
  | vip_network_id      | 8e748892-365d-4fa9-ac6d-f71f608db340 |
  | vip_port_id         | f42f085f-734c-43ae-a570-cd2c84f13abc |
  | vip_qos_policy_id   | None                                 |
  | vip_subnet_id       | d6a914ad-6331-4028-bf8b-3a22f6c20f57 |
  +---------------------+--------------------------------------+


After a few minutes the provisioning status should change from `PENDING_CREATE` to `ACTIVE`.
Then, you can associate a floating IP in order to access the load balancer externally.

To create a load balancer which is to balance incoming traffic from the External network:

.. code-block:: bash

  openstack loadbalancer create --name external-lb --vip-subnet-id External


This will create a load balancer and create a floating IP to be associated with it.


Listener
~~~~~~~~~

Listeners are set up and attached to load balancers to 'listen' for incoming traffic trying to connect to a pool through the load balancer. Listeners can be created using:

.. code-block:: text

  openstack loadbalancer listener create [-h] [-f {json,shell,table,value,yaml}] [-c COLUMN] [--noindent] [--prefix PREFIX]
                                                [--max-width <integer>] [--fit-width] [--print-empty] [--name <name>]
                                                [--description <description>] --protocol {TCP,HTTP,HTTPS,TERMINATED_HTTPS,UDP}
                                                [--connection-limit <limit>] [--default-pool <pool>]
                                                [--default-tls-container-ref <container_ref>]
                                                [--sni-container-refs [<container_ref> [<container_ref> ...]]]
                                                [--insert-headers <header=value,...>] --protocol-port <port>
                                                [--timeout-client-data <timeout>] [--timeout-member-connect <timeout>]
                                                [--timeout-member-data <timeout>] [--timeout-tcp-inspect <timeout>]
                                                [--enable | --disable] [--client-ca-tls-container-ref <container_ref>]
                                                [--client-authentication {NONE,OPTIONAL,MANDATORY}]
                                                [--client-crl-container-ref <client_crl_container_ref>]
                                                [--allowed-cidr [<allowed_cidr>]] [--wait] [--tls-ciphers <tls_ciphers>]
                                                [--tls-version [<tls_versions>]]
                                                <loadbalancer>

  Create a listener

  positional arguments:
    <loadbalancer>        Load balancer for the listener (name or ID).

  optional arguments:
    -h, --help            show this help message and exit
    --name <name>         Set the listener name.
    --description <description>
                          Set the description of this listener.
    --protocol {TCP,HTTP,HTTPS,TERMINATED_HTTPS,UDP}
                          The protocol for the listener.
    --connection-limit <limit>
                          Set the maximum number of connections permitted for this listener.
    --default-pool <pool>
                          Set the name or ID of the pool used by the listener if no L7 policies match.
    --default-tls-container-ref <container_ref>
                          The URI to the key manager service secrets container containing the certificate and key for TERMINATED_TLS
                          listeners.
    --sni-container-refs [<container_ref> [<container_ref> ...]]
                          A list of URIs to the key manager service secrets containers containing the certificates and keys for
                          TERMINATED_TLS the listener using Server Name Indication.
    --insert-headers <header=value,...>
                          A dictionary of optional headers to insert into the request before it is sent to the backend member.
    --protocol-port <port>
                          Set the protocol port number for the listener.
    --timeout-client-data <timeout>
                          Frontend client inactivity timeout in milliseconds. Default: 50000.
    --timeout-member-connect <timeout>
                          Backend member connection timeout in milliseconds. Default: 5000.
    --timeout-member-data <timeout>
                          Backend member inactivity timeout in milliseconds. Default: 50000.
    --timeout-tcp-inspect <timeout>
                          Time, in milliseconds, to wait for additional TCP packets for content inspection. Default: 0.
    --enable              Enable listener (default).
    --disable             Disable listener.
    --client-ca-tls-container-ref <container_ref>
                          The URI to the key manager service secrets container containing the CA certificate for TERMINATED_TLS
                          listeners.
    --client-authentication {NONE,OPTIONAL,MANDATORY}
                          The TLS client authentication verify options for TERMINATED_TLS listeners.
    --client-crl-container-ref <client_crl_container_ref>
                          The URI to the key manager service secrets container containting the CA revocation list file for
                          TERMINATED_TLS listeners.
    --allowed-cidr [<allowed_cidr>]
                          CIDR to allow access to the listener (can be set multiple times).
    --wait                Wait for action to complete
    --tls-ciphers <tls_ciphers>
                          Set the TLS ciphers to be used by the listener in OpenSSL format.
    --tls-version [<tls_versions>]
                          Set the TLS protocol version to be used by the listener (can be set multiple times).



Below is an example of creating a listener for the load loadbalancer ``test-lb`` which runs a HTTP protocol on port 80. This means any connections to the load balancer have to be done via this port as defined by the listener.

.. code-block:: bash

  openstack loadbalancer listener create --name test-listener --protocol HTTP --protocol-port 80 test-lb

  #This should return a table with details of the listener

  +-----------------------------+--------------------------------------+
  | Field                       | Value                                |
  +-----------------------------+--------------------------------------+
  | admin_state_up              | True                                 |
  | connection_limit            | -1                                   |
  | created_at                  | 2020-11-16T09:10:22                  |
  | default_pool_id             | None                                 |
  | default_tls_container_ref   | None                                 |
  | description                 |                                      |
  | id                          | ec01a05f-54be-419f-84cf-e89bba7e6acc |
  | insert_headers              | None                                 |
  | l7policies                  |                                      |
  | loadbalancers               | fe22e256-d409-4f91-867f-508d08566892 |
  | name                        | test-listener                        |
  | operating_status            | OFFLINE                              |
  | project_id                  | PROJECT_ID                           |
  | protocol                    | HTTP                                 |
  | protocol_port               | 80                                   |
  | provisioning_status         | PENDING_CREATE                       |
  | sni_container_refs          | []                                   |
  | timeout_client_data         | 50000                                |
  | timeout_member_connect      | 5000                                 |
  | timeout_member_data         | 50000                                |
  | timeout_tcp_inspect         | 0                                    |
  | updated_at                  | None                                 |
  | client_ca_tls_container_ref | None                                 |
  | client_authentication       | NONE                                 |
  | client_crl_container_ref    | None                                 |
  | allowed_cidrs               | None                                 |
  | tls_ciphers                 |                                      |
  | tls_versions                |                                      |
  +-----------------------------+--------------------------------------+


> Listeners can refer to several pools.

Pool and Members
~~~~~~~~~~~~~~~~~

A pool consists of members that serve traffic behind a load balancer, each member has a specified IP address and port to serve traffic.

The following load balancer algorithms are supported:
  - **ROUND_ROBIN:** each member of the pool is used in turn to handle incoming traffic.
  - **LEAST_CONNECTIONS:**  a VM with the least number of connections is selected to receive the next incoming connection.
  - **SOURCE_IP and SOURCE_IP_PORT:** Source IP is hashed and divided by the total weight of the active VMs to determine which VM will receive the request.

    **NOTE:** Pools can only be associated with one listener.
    **NOTE:** The recommended and preferred algorithm is SOURCE_IP

Pools are created using ``openstack loadbalancer pool create``:

.. code-block:: text

  openstack loadbalancer pool create [-h] [-f {json,shell,table,value,yaml}] [-c COLUMN] [--noindent] [--prefix PREFIX]
                                            [--max-width <integer>] [--fit-width] [--print-empty] [--name <name>]
                                            [--description <description>] --protocol {TCP,HTTP,HTTPS,TERMINATED_HTTPS,PROXY,UDP}
                                            (--listener <listener> | --loadbalancer <load_balancer>)
                                            [--session-persistence <session persistence>] --lb-algorithm
                                            {SOURCE_IP,ROUND_ROBIN,LEAST_CONNECTIONS,SOURCE_IP_PORT} [--enable | --disable]
                                            [--tls-container-ref <container-ref>] [--ca-tls-container-ref <ca_tls_container_ref>]
                                            [--crl-container-ref <crl_container_ref>] [--enable-tls | --disable-tls] [--wait]
                                            [--tls-ciphers <tls_ciphers>] [--tls-version [<tls_versions>]]

  optional arguments:
    -h, --help            show this help message and exit
    --name <name>         Set pool name.
    --description <description>
                          Set pool description.
    --protocol {TCP,HTTP,HTTPS,TERMINATED_HTTPS,PROXY,UDP}
                          Set the pool protocol.
    --listener <listener>
                          Listener to add the pool to (name or ID).
    --loadbalancer <load_balancer>
                          Load balncer to add the pool to (name or ID)
    --session-persistence <session persistence>
                          Set the session persistence for the listener (key=value).
    --lb-algorithm {SOURCE_IP,ROUND_ROBIN,LEAST_CONNECTIONS,SOURCE_IP_PORT}
                          Load balancing algorithm to use.
    --enable              Enable pool (default).
    --disable             Disable pool.
    --tls-container-ref <container-ref>
                          The reference to the key manager service secrets container containing the certificate and key for
                          ``tls_enabled`` pools to re-encrpt the traffic to backend member servers.
    --ca-tls-container-ref <ca_tls_container_ref>
                          The reference to the key manager service secrets container containing the CA certificate for
                          ``tls_enabled`` pools to check the backend member servers certificates
    --crl-container-ref <crl_container_ref>
                          The reference to the key manager service secrets container containting the CA revocation list file for
                          ``tls_enabled`` pools to validate the backend member servers certificates.
    --enable-tls          Enable backend member re-encryption.
    --disable-tls         Disable backend member re-encryption.
    --wait                Wait for action to complete
    --tls-ciphers <tls_ciphers>
                          Set the TLS ciphers to be used by the pool in OpenSSL cipher string format.
    --tls-version [<tls_versions>]
                          Set the TLS protocol version to be used by the pool (can be set multiple times).

Then members can be defined for the pool using `openstack loadbalancer member create`:

.. code-block:: text

  openstack loadbalancer member create [-h] [-f {json,shell,table,value,yaml}] [-c COLUMN] [--noindent] [--prefix PREFIX]
                                              [--max-width <integer>] [--fit-width] [--print-empty] [--name <name>]
                                              [--disable-backup | --enable-backup] [--weight <weight>] --address <ip_address>
                                              [--subnet-id <subnet_id>] --protocol-port <protocol_port>
                                              [--monitor-port <monitor_port>] [--monitor-address <monitor_address>]
                                              [--enable | --disable] [--wait]
                                              <pool>

  Creating a member in a pool

  positional arguments:
    <pool>                ID or name of the pool to create the member for.

  optional arguments:
    -h, --help            show this help message and exit
    --name <name>         Name of the member.
    --disable-backup      Disable member backup (default)
    --enable-backup       Enable member backup
    --weight <weight>     The weight of a member determines the portion of requests or connections it services compared to the other
                          members of the pool.
    --address <ip_address>
                          The IP address of the backend member server
    --subnet-id <subnet_id>
                          The subnet ID the member service is accessible from.
    --protocol-port <protocol_port>
                          The protocol port number the backend member server is listening on.
    --monitor-port <monitor_port>
                          An alternate protocol port used for health monitoring a backend member.
    --monitor-address <monitor_address>
                          An alternate IP address used for health monitoring a backend member.
    --enable              Enable member (default)
    --disable             Disable member
    --wait                Wait for action to complete



Example
^^^^^^^^

.. code-block:: bash

  #Use openstack loadbalancer pool create to define the pool which is the default pool for test-listener
  openstack loadbalancer pool create --name test-pool --lb-algorithm ROUND_ROBIN --listener ec01a05f-54be-419f-84cf-e89bba7e6acc --protocol HTTP

  #This should return a table with details of the pool

  +----------------------+--------------------------------------+
  | Field                | Value                                |
  +----------------------+--------------------------------------+
  | admin_state_up       | True                                 |
  | created_at           | 2020-11-16T09:16:31                  |
  | description          |                                      |
  | healthmonitor_id     |                                      |
  | id                   | 733b78a8-4e0d-4d31-9eb4-e8084a94dac0 |
  | lb_algorithm         | ROUND_ROBIN                          |
  | listeners            | ec01a05f-54be-419f-84cf-e89bba7e6acc |
  | loadbalancers        | fe22e256-d409-4f91-867f-508d08566892 |
  | members              |                                      |
  | name                 | test-pool                            |
  | operating_status     | OFFLINE                              |
  | project_id           | PROJECT_ID                           |
  | protocol             | HTTP                                 |
  | provisioning_status  | PENDING_CREATE                       |
  | session_persistence  | None                                 |
  | updated_at           | None                                 |
  | tls_container_ref    | None                                 |
  | ca_tls_container_ref | None                                 |
  | crl_container_ref    | None                                 |
  | tls_enabled          | False                                |
  | tls_ciphers          |                                      |
  | tls_versions         |                                      |
  +----------------------+--------------------------------------+

  #Then we can define the pool members using the command 'openstack loadbalancer member create'
  openstack loadbalancer member create --subnet-id <private-subnet-id> --address <server-ip> --protocol_port 80 test-pool



After setting up the load balancer, listener, and pool we can test that we can access the VM in the pool via the load balancer:

.. code-block:: bash

  ssh <user>@<lb-floating-IP> -p <listener-port>


More load balancing examples can be found here: https://docs.openstack.org/octavia/train/user/guides/basic-cookbook.html

Load Balancer Status
~~~~~~~~~~~~~~~~~~~~~

We can also check the status of a load balancing pool from the command line. For example, we can check the status of a load balancer that is in front of a kubernetes service being hosted on multiple nodes.

To check the load balancer status we can use the following command:

.. code-block:: bash

  openstack loadbalancer status show <loadbalancer-id>

  #This will give a load balancer status in the form similar to the following:

  {
      "loadbalancer": {
          "listeners": [
              {
                  "pools": [
                      {
                          "name": "kubernetes-pool-t3ue6p7zvss5",
                          "health_monitor": {
                              "provisioning_status": "ACTIVE",
                              "type": "TCP",
                              "id": "6137160c-7835-48f8-b79a-214359f48a92",
                              "operating_status": "ONLINE",
                              "name": ""
                          },
                          "provisioning_status": "ACTIVE",
                          "members": [
                              {
                                  "name": "cluster-node-0",
                                  "provisioning_status": "ACTIVE",
                                  "address": "10.0.0.131",
                                  "protocol_port": 30858,
                                  "id": "edb62670-8b79-4912-b449-6269f0ff3076",
                                  "operating_status": "OFFLINE"
                              },
                              {
                                  "name": "cluster-node-1",
                                  "provisioning_status": "ACTIVE",
                                  "address": "10.0.0.8",
                                  "protocol_port": 30858,
                                  "id": "564e4f40-9e2c-4eb2-877b-ce4169b4faa9",
                                  "operating_status": "OFFLINE"
                              }
                          ],
                          "id": "1cef96d9-d429-413b-a92f-daf2bc0d419d",
                          "operating_status": "OFFLINE"
                      }
                  ],
                  "provisioning_status": "ACTIVE",
                  "id": "4ff15be3-71f8-475e-81ea-5dfcb1b92e0f",
                  "operating_status": "OFFLINE",
                  "name": "kubernetes-listener-tnroqyfm5ruw"
              }
          ],
          "provisioning_status": "ACTIVE",
          "id": "56c1563f-9afc-4deb-9ba4-ea99e9b71547",
          "operating_status": "OFFLINE",
          "name": "kubernetes-lb-yvawgzx7acca"
      }
  }



Health Monitor
~~~~~~~~~~~~~~~

It is possible to set up a listener without a health monitor, however OpenStack recommends to always configure production load balancers with a health monitor. Health monitors in are attached to load balancer pools and monitors the health of each pool member. Should a pool member fail a health check, the health monitor should remove that member temporarily from the pool. If that pool member begins to respond to health checks, the health monitor returns the member to the pool.


**Health Monitor Options**

  - **delay:** number of seconds to check between health checks

  - **timeout:** the number of seconds to any given health check to complete. Timeout should always be smaller than delay.

  -**max-retries:** number of subsequent health checks a given back-end server must fail before it is considered down, or that a failed back-end server must pass to be considered up again.


**HTTP Health Monitors**

By default, Octavia will probe the "/" path on the application server. However, in many applications this is not appropriate because the "/" path ends up being a cached page, or causes the server to do more work than it is necessary for a basic health check.

HTTP health monitors also have the following options:

  - **url_path:** path part of the URL that should be retrieved form the back end server. By default it is "/".

  - **http_method:** HTTP method that should be used to retrieve the url_path. By default this is "GET".
  - **expected_codes:** list of HTTP status codes that indicate an OK health check. By default this is just "200".

To create health monitors and attach them to load balancing pools, we can use the command:

.. code-block:: text

  openstack loadbalancer healthmonitor create [-h] [-f {json,shell,table,value,yaml}] [-c COLUMN] [--noindent]
                                                     [--prefix PREFIX] [--max-width <integer>] [--fit-width] [--print-empty]
                                                     [--name <name>] --delay <delay> [--domain-name <domain_name>]
                                                     [--expected-codes <codes>]
                                                     [--http-method {GET,POST,DELETE,PUT,HEAD,OPTIONS,PATCH,CONNECT,TRACE}]
                                                     [--http-version <http_version>] --timeout <timeout> --max-retries <max_retries>
                                                     [--url-path <url_path>] --type {PING,HTTP,TCP,HTTPS,TLS-HELLO,UDP-CONNECT}
                                                     [--max-retries-down <max_retries_down>] [--enable | --disable] [--wait]
                                                     <pool>

  #positional argument
  <pool> define pool for the health monitor

  #optional arguments:
  --name <name>         Set the health monitor name.
  --delay <delay>       Set the time in seconds, between sending probes to members.
  --domain-name <domain_name>
                        Set the domain name, which be injected into the HTTP Host Header to the backend server for HTTP health
                        check.
  --expected-codes <codes>
                        Set the list of HTTP status codes expected in response from the member to declare it healthy.
  --http-method {GET,POST,DELETE,PUT,HEAD,OPTIONS,PATCH,CONNECT,TRACE}
                        Set the HTTP method that the health monitor uses for requests.
  --http-version <http_version>
                        Set the HTTP version.
  --timeout <timeout>   Set the maximum time, in seconds, that a monitor waits to connect before it times out. This value must be
                        less than the delay value.
  --max-retries <max_retries>
                        The number of successful checks before changing the operating status of the member to ONLINE.
  --url-path <url_path>
                        Set the HTTP URL path of the request sent by the monitor to test the health of a backend member.
  --type {PING,HTTP,TCP,HTTPS,TLS-HELLO,UDP-CONNECT}
                        Set the health monitor type.
  --max-retries-down <max_retries_down>
                        Set the number of allowed check failures before changing the operating status of the member to ERROR.
  --enable              Enable health monitor (default).
  --disable             Disable health monitor.
  --wait                Wait for action to complete




Status Codes
------------

The load balancer, listener, health monitor, pool and pool member have a provisioning status and an operating status.

Provisioning Status Codes
~~~~~~~~~~~~~~~~~~~~~~~~~~

.. list-table::
  :header-rows: 1

  * - Code
    - Reason
  * - **ACTIVE**
    - Entity Provisioned successfully
  * - **DELETED**
    - Entity successfully deleted
  * - **ERROR** Please refer to error messages
    - Provisioning failed
  * - **PENDING_CREATE**
    - Entity is being created
  * - **PENDING_UPDATE**
    - Entity is being updated
  * - **PENDING_DELETE**
    - Entity is being deleted


Operation Status Codes
~~~~~~~~~~~~~~~~~~~~~~~~

.. list-table::
  :header-rows: 1

  * - Status
    - Reason
  * - **ONLINE**
    - Entity is operating normally. All Pool members are healthy
  * - **DRAINING**
    - Member is not accepting new connections
  * - **OFFLINE**
    - Entity is administratively disabled
  * - **DEGRADED**
    - One or more components are in ERROR
  * - **ERROR** Please refer to error messages
    - Entity has failed. Member is failing health  monitoring check. All pool members are in ERROR
  * - **NO_MONITOR**
    - No health monitor is configured. Current status in known




References
-----------


https://docs.openstack.org/octavia/train/reference/introduction.html

https://docs.openstack.org/octavia/train/user/guides/basic-cookbook.html

https://docs.openstack.org/octavia/train/reference/glossary.html

https://wiki.openstack.org/wiki/Neutron/LBaaS/Deprecation

http://cbonte.github.io/haproxy-dconv/1.8/configuration.html#4-balance

https://docs.openstack.org/api-ref/load-balancer/v2/index.html?expanded=create-pool-detail,show-pool-details-detail#general-api-overview

https://docs.openstack.org/mitaka/networking-guide/config-lbaas.html
