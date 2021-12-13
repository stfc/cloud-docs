OpenStack Magnum: Container-as-a-Service
========================================

    **Important:** Magnum is currently in technical preview on STFC Cloud. If you have any feedback or suggestions please send it to cloud-support@gridpp.rl.ac.uk

Magnum is the Container-as-a-Service for OpenStack and can be used to create and launch clusters.

The clusters Magnum supports are:

- Kubernetes
- Swarm
- Mesos

Magnum uses the following OpenStack Components:

- **Keystone:** for multi tenancy
- **Nova compute:** computing service
- **Heat:** virtual application deployment service
- **Neutron:** networking
- **Glance:** virtual machine image
- **Cinder:** volume service

Magnum uses Cluster Templates to define the desired cluster, and passes the template for
the cluster to Heat to create the user's cluster.


python-magnumclient
--------------------

To use commands for creating or managing clusters using Magnum in the command line, you will
need to install the Magnum CLI. This can be done using pip:

.. code-block:: bash

  pip install python-magnumclient


To test that we can run OpenStack commands for container orchestration engines (coe), we can run the following command:

.. code-block:: bash

  openstack coe cluster list #list clusters in the project

This should return an empty line if there are no clusters in the project, or a table similar to the following:

.. code-block:: bash

  +----------+---------------------------+----------------+------------+--------------+-----------------+---------------+
  | uuid     | name                      | keypair        | node_count | master_count | status          | health_status |
  +----------+---------------------------+----------------+------------+--------------+-----------------+---------------+
  | UUID_1   | test-1                    | mykeypair      |          1 |            1 | CREATE_COMPLETE | UNKNOWN       |
  | UUID_2   | kubernetes-cluster-test-2 | mykeypair      |          1 |            1 | CREATE_COMPLETE | UNKNOWN       |
  +----------+---------------------------+----------------+------------+--------------+-----------------+---------------+


Other commands available from ``python-magumclient`` include:

.. code-block:: bash

  openstack coe <commands>

  coe cluster config # download cluster config to current directory
  coe cluster create # create a cluster
  coe cluster delete # delete a cluster
  coe cluster list # list clusters
  coe cluster resize # resize cluster
  coe cluster show # view details of cluster
  coe cluster template create # create a cluster template
  coe cluster template delete # delete a cluster template - can only be deleted if there are no clusters using the template
  coe cluster template list # list templates
  coe cluster template show # view details of cluster template
  coe cluster template update # update a cluster template
  coe cluster update # update a cluster e.g node count
  coe cluster upgrade # upgrade a cluster e.g upgrade Kubernetes version in cluster
  coe nodegroup create # create a nodegroup
  coe nodegroup delete # delete a nodegroup
  coe nodegroup list # list nodegroups in cluster
  coe nodegroup show # view details of a nodegroup in a cluster
  coe nodegroup update # update a nodegroup




To view details of a cluster:

.. code-block:: bash

  openstack coe cluster view <cluster-uuid>

  #This should return a table similar to the table below:

  +----------------------+---------------------------------------------------------------------------------------------------------------------------------------------------------------+
  | Field                | Value                                                                                                                                                         |
  +----------------------+---------------------------------------------------------------------------------------------------------------------------------------------------------------+
  | status               | CREATE_COMPLETE                                                                                                                                               |
  | health_status        | UNKNOWN                                                                                                                                                       |
  | cluster_template_id  | e186c6e2-dd47-4df0-ac3f-3eb46e64cb3d                                                                                                                          |
  | node_addresses       | ['10.0.0.163']                                                                                                                                                |
  | uuid                 | 27cdcad8-375f-4d4f-a186-8fa99b80c5c5                                                                                                                          |
  | stack_id             | e881d058-db91-4de6-9527-193eecebd05d                                                                                                                          |
  | status_reason        | None                                                                                                                                                          |
  | created_at           | 2020-09-07T15:39:32+00:00                                                                                                                                     |
  | updated_at           | 2020-09-07T15:52:54+00:00                                                                                                                                     |
  | coe_version          | v1.14.3                                                                                                                                                       |
  | labels               | {'auto_healing': 'true', 'kube_tag': 'v1.14.3', 'heat_container_agent_tag': 'train-stable-3', 'kube_dashboard_enabled': '1', 'ingress_controller': 'traefik'} |
  | labels_overridden    |                                                                                                                                                               |
  | labels_skipped       |                                                                                                                                                               |
  | labels_added         |                                                                                                                                                               |
  | fixed_network        | None                                                                                                                                                          |
  | fixed_subnet         | None                                                                                                                                                          |
  | floating_ip_enabled  | False                                                                                                                                                         |
  | faults               |                                                                                                                                                               |
  | keypair              | mykeypair                                                                                                                                                     |
  | api_address          | https://10.0.0.212:6443                                                                                                                                       |
  | master_addresses     | ['10.0.0.212']                                                                                                                                                |
  | master_lb_enabled    |                                                                                                                                                               |
  | create_timeout       | 60                                                                                                                                                            |
  | node_count           | 1                                                                                                                                                             |
  | discovery_url        | https://discovery.etcd.io/31c1d9cf44cf4fda5710946d57980bb1                                                                                                    |
  | master_count         | 1                                                                                                                                                             |
  | container_version    | 1.12.6                                                                                                                                                        |
  | name                 | kubernetes-cluster-test-1                                                                                                                                     |
  | master_flavor_id     | c1.medium                                                                                                                                                     |
  | flavor_id            | c1.medium                                                                                                                                                     |
  | health_status_reason | {'api': 'The cluster kubernetes-cluster-test-1 is not accessible.'}                                                                                           |
  | project_id           | PROJECT_ID                                                                                                                                                    |
  +----------------------+---------------------------------------------------------------------------------------------------------------------------------------------------------------+

To view the list of cluster templates:

.. code-block:: bash

  openstack coe cluster template list

  # This should return a list of cluster templates in the project

  +--------------------------------------+------------------------------+
  | uuid                                 | name                         |
  +--------------------------------------+------------------------------+
  | e186c6e2-dd47-4df0-ac3f-3eb46e64cb3d | kubernetes-v1_14_3           |
  | 0bd1232d-06f2-42ca-b6d5-c27e57f26c3c | kubernetes-ha-master-v1_14_3 |
  | a07903d0-aecf-4f15-a35f-f4fd74060e2f | coreos-kubernetes-v1_14_3    |
  +--------------------------------------+------------------------------+

To view the details of a specific template:

.. code-block:: bash

  openstack coe cluster template show <cluster-template-uuid>

  #This will return a table similar to:

  +-----------------------+---------------------------------------------------------------------------------------------------------------------------------------------------------------+
  | Field                 | Value                                                                                                                                                         |
  +-----------------------+---------------------------------------------------------------------------------------------------------------------------------------------------------------+
  | insecure_registry     | -                                                                                                                                                             |
  | labels                | {'kube_tag': 'v1.14.3', 'kube_dashboard_enabled': '1', 'heat_container_agent_tag': 'train-stable-3', 'auto_healing': 'true', 'ingress_controller': 'traefik'} |
  | updated_at            | -                                                                                                                                                             |
  | floating_ip_enabled   | False                                                                                                                                                         |
  | fixed_subnet          | -                                                                                                                                                             |
  | master_flavor_id      | c1.medium                                                                                                                                                     |
  | uuid                  | e186c6e2-dd47-4df0-ac3f-3eb46e64cb3d                                                                                                                          |
  | no_proxy              | -                                                                                                                                                             |
  | https_proxy           | -                                                                                                                                                             |
  | tls_disabled          | False                                                                                                                                                         |
  | keypair_id            | -                                                                                                                                                             |
  | public                | True                                                                                                                                                          |
  | http_proxy            | -                                                                                                                                                             |
  | docker_volume_size    | 3                                                                                                                                                             |
  | server_type           | vm                                                                                                                                                            |
  | external_network_id   | External                                                                                                                                                      |
  | cluster_distro        | fedora-atomic                                                                                                                                                 |
  | image_id              | cf37f7d0-1d6b-4aab-a23b-df58542c59cb                                                                                                                          |
  | volume_driver         | -                                                                                                                                                             |
  | registry_enabled      | False                                                                                                                                                         |
  | docker_storage_driver | devicemapper                                                                                                                                                  |
  | apiserver_port        | -                                                                                                                                                             |
  | name                  | kubernetes-v1_14_3                                                                                                                                            |
  | created_at            | 2020-09-07T07:17:13+00:00                                                                                                                                     |
  | network_driver        | flannel                                                                                                                                                       |
  | fixed_network         | -                                                                                                                                                             |
  | coe                   | kubernetes                                                                                                                                                    |
  | flavor_id             | c1.medium                                                                                                                                                     |
  | master_lb_enabled     | False                                                                                                                                                         |
  | dns_nameserver        | 8.8.8.8                                                                                                                                                       |
  | hidden                | False                                                                                                                                                         |
  +-----------------------+---------------------------------------------------------------------------------------------------------------------------------------------------------------+


To delete a cluster or cluster template:

    **Note:** Cluster Templates can only be deleted if there are no clusters using the template.


.. code-block:: bash

  # To delete a template
  openstack coe cluster template delete <cluster-template-id>

  # To delete a cluster
  openstack coe cluster delete <cluster-id>


Creating Clusters
-----------------

Clusters can be created using:

- OpenStack CLI
- Horizon Web UI
- Heat Templates: using the resources ``OS::Magnum::ClusterTemplate`` and ``OS::Magnum::Cluster``

    The documentation **Create A Kubernetes Cluster** has examples for handling cluster templates and creating a Kubernetes cluster in the command line.

Create a Cluster using OpenStack CLI
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Create A Cluster Template
^^^^^^^^^^^^^^^^^^^^^^^^^

To create a cluster template, we can use the following command:

.. code-block:: bash

   openstack coe cluster template create [-h] [-f {json,shell,table,value,yaml}] [-c COLUMN] [--noindent] [--prefix PREFIX]
                                             [--max-width <integer>] [--fit-width] [--print-empty] --coe <coe> --image <image>
                                             --external-network <external-network> [--keypair <keypair>]
                                             [--fixed-network <fixed-network>] [--fixed-subnet <fixed-subnet>]
                                             [--network-driver <network-driver>] [--volume-driver <volume-driver>]
                                             [--dns-nameserver <dns-nameserver>] [--flavor <flavor>]
                                             [--master-flavor <master-flavor>] [--docker-volume-size <docker-volume-size>]
                                             [--docker-storage-driver <docker-storage-driver>] [--http-proxy <http-proxy>]
                                             [--https-proxy <https-proxy>] [--no-proxy <no-proxy>]
                                             [--labels <KEY1=VALUE1,KEY2=VALUE2;KEY3=VALUE3...>] [--tls-disabled] [--public]
                                             [--registry-enabled] [--server-type <server-type>] [--master-lb-enabled]
                                             [--floating-ip-enabled] [--floating-ip-disabled] [--hidden] [--visible]
                                             <name>

`<name>`: Name of the ClusterTemplate to create. The name does not have to be unique but the template UUID should be used to select a ClusterTemplate if more than one template has the same name.

`<coe>`: Container Orchestration Engine to use. Supported drivers are: **kubernetes, swarm, mesos**.

`<image>`: Name or UUID of the base image to boot servers for the clusters.

**Images which OpenStack Magnum supports:**

.. list-table::
   :header-rows: 1

   * - COE
     - os_distro
   * - Kubernetes
     - fedora-atomic, coreos
   * - Swarm
     - fedora-atomic
   * - Mesos
     - ubuntu

``<keypair>``: SSH keypair to configure in servers for ssh access. The login name is specific to the cluster driver.
  - fedora-atomic: ``ssh -i <private-key> fedora@<ip-address>``
  - coreos: ``ssh -i <private-key> core@<ip-address>``

``external-network <external-network>``: name or ID of a Neutron network to provide connectivity to the external internet.

``--public`` Access to a ClusterTemplate is, by default, limited to admin, owner or users within the same tenant as the owners. Using this flag makes the template accessible by other users. Default is not public

``server-type <server-type>``: Servers can be VM or bare metal (bm). The default is vm.

``network-driver <network-driver>`` Name of a network driver for providing networks for the containers - this is different and separate from the Neutron network for the cluster. Drivers that Magnum supports:

.. list-table::
   :header-rows: 1

   * - COE
     - Network Driver
     - Default
   * - Kubernetes
     - flannel, calico
     - flannel
   * - Swarm
     - docker, flannel
     - flannel
   * - Mesos
     - docker
     - docker

**Note:** For Kubernetes clusters, we are using the ``flannel`` network driver.

``dns-nameserver <dns-nameserver>``: The DNS nameserver for the servers and containers in the cluster
to use. The default is 8.8.8.8.

``flavor <flavor>``: flavor to use for worker nodes. The default is m1.small. Can be overridden at cluster creation.

``master-flavor <master-flavor>``: flavor for master nodes. Default is m1.small. Can be overridden at cluster creation.

``http-proxy <http-proxy>``: The IP address for a proxy to use when direct http access from the servers to
sites on the external internet is blocked. The format is a URL including a
port number. The default is None.

``https-proxy <https-proxy>``: The IP address for a proxy to use when direct https access from the servers
to sites on the external internet is blocked. The format is a URL including a
port number. The default is None.

``no-proxy <no-proxy>``: When a proxy server is used, some sites should not go through the proxy and
should be accessed normally. In this case, you can specify these sites as a comma separated list of
IPs. The default is None.

``docker-volume-size <docker-volume-size>``: If specified, container images will be stored in a cinder
volume of the specified size in GB. Each cluster node will have a volume attached of the above
size. If not specified, images will be stored in the compute instances local disk. For the devicemapper storage driver, must specify volume and the minimum value is 3GB. For the overlay
and overlay2 storage driver, the minimum value is 1GB or None(no volume). This value can be
overridden at cluster creation.

``docker-storage-driver <docker-storage-driver>``: The name of a driver to manage the storage for the
images and the containers writable layer. The default is devicemapper.

``labels <KEY1=VALUE1,KEY2=VALUE2;KEY3=VALUE3>``: Arbitrary labels in the form of
key=value pairs. The accepted keys and valid values are defined in the cluster drivers. They
are used as a way to pass additional parameters that are specific to a cluster driver. The value can be
overridden at cluster creation.

``--tls-disabled`` Transport Layer Security (TLS) is normally enabled to secure the cluster. The default is TLS enabled.

``--registry-enabled`` Docker images by default are pulled from the public Docker registry,
but in some cases, users may want to use a private registry. This option
provides an alternative registry based on the Registry V2: Magnum
will create a local registry in the cluster backed by swift to host the
images. Refer to Docker Registry 2.0 for more details. The default is
to use the public registry.

``--master-lb-enabled`` Since multiple masters may exist in a cluster, a load balancer is created to provide the API endpoint for the cluster and to direct requests
to the masters. As we have Octavia enabled, Octavia would create these load balancers. The default is master load balancers are created.


Create a Cluster
^^^^^^^^^^^^^^^^^

We can create clusters using a cluster template from our template list.
To create a cluster, we use the command:

.. code-block:: bash

  openstack coe cluster create  --cluster-template <cluster-template>
                                --discovery-url <discovery-url>
                                --master-count <master-count>
                                --node-count <node-count>
                                --timeout <timeout>
                                --merge-labels
                                --master-lb-enabled
                                #The following options can be used to overwrite the same options in the cluster template
                                --docker-volume-size <docker-volume-size>
                                --labels <KEY1=VALUE1,KEY2=VALUE2;KEY3=VALUE3...>
                                --keypair <keypair>
                                --master-flavor <master-flavor>
                                --flavor <flavor>
                                --fixed-network <fixed-network>
                                --fixed-subnet <fixed-subnet>
                                --floating-ip-enabled
                                --floating-ip-disabled
                                # To add labels to use with the template labels, we can use:
                                --merge-labels
                                <name>


**Note:** It is recommended that to have master load balancers enabled, to use the kubernetes-ha-master-v1_14_3 template,
or create a new cluster template and include the flag ``--master-lb-enabled``.

Labels
^^^^^^^^

Labels are used by OpenStack Magnum to define a range of parameters such as the Kubernetes version, enable autoscaling, enable autohealing, version of draino to use etc. Any labels included at cluster creation overwrite the labels in the cluster template.
A table containing all of the labels which Magnum uses can be found here:

https://docs.openstack.org/magnum/train/user/

    **Note:** For OpenStack Train release, Magnum only offers labels for installing Helm 2 and Tiller. However, Helm 3 can be installed onto the master node after the cluster has been created.


Horizon Web Interface
~~~~~~~~~~~~~~~~~~~~~~

Clusters can also be created using the Horizon Web Interface. Clusters and their templates can be found under the ``Container Infra`` section.


    There are a few differences between the parameters which can be defined when creating a cluster using the CLI or Horizon Web UI.
    If you are using the Horizon web UI to create clusters, the **fixed network, fixed subnet, and floating ip enabled** can only be defined in the cluster template.


Heat Templates
~~~~~~~~~~~~~~

Clusters can also be created using a Heat template using the resources ``OS::Magnum::CluterTemplate`` and ``OS::Magnum::Cluster``.

    This will instruct Heat to pass the resources to Magnum, which will pass a stack template to Heat to create a cluster - so two stacks are built in total.


OS::Magnum::ClusterTemplate
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: yaml

  resources:
    cluster_template:
        type: OS::Magnum::ClusterTemplate
        properties:
          #required
          coe: String # Container Orchestration Engine: kubernetes, swarm, mesos
          external_network: String # External neutron network or UUID to attach the cluster
          image: String # The image name/UUID to use as a base image for the cluster
          # optional
          dns_nameserver: String # DNS nameserver address, must be of type ip_addr
          docker_storage_driver: String # Docker storage driver: devicemapper, overlay
          docker_volume_size: Integer # Size in GB of docker volume, must be at least 1
          fixed_network: String # The fixed neutron network name or UUID to attach the Cluster
          fixed_subnet: String # The fixed neutron subnet name or UUID to attach the Cluster
          flavor: String # Flavor name or UUID to use when launching a cluster
          floating_ip_enabled: Boolean # True by default, determines whether a cluster should have floating IPs
          http_proxy: String # http_proxy address to use for nodes in cluster
          https_proxy: String # https_proxy address to use for nodes in cluster
          keypair: String # SSH keypair to load into cluster nodes
          labels: {...} # labels in form of key=value pairs to associate with cluster
          master_flavor: String # flavor name or UUID to associate with the master node
          master_lb_enabled: Boolean # Defaults to true. Determines whether there should be a load balancer for master nodes
          name: String # Template name
          network_driver: String # Name of driver to use for instantiating container networks. Magnum uses pre-configured driver for specific COE by default
          no_proxy: String # A comma separated list of addresses for which proxies should not be used in the cluster
          public: Boolean # Defaults to false. True makes the cluster template public. Must have the permissions to publish templates in Magnum
          registry_enabled: Boolean # Defaults to false. Enable registry in the cluster
          server_type: String # Define server type to use. Defaults to vm. Allowed: vm, bm
          tls_disabled: Boolean # Disable TLS in the Cluster. Defaults to false
          volume_driver: String # Volume driver name for instantiating container volume. Allowed: cinder, rexray

  outputs:

    detailed-information:
      description: Detailed Information about the resource
      value: {get_attr: [cluster_template, show]}


OS::Magnum::Cluster
^^^^^^^^^^^^^^^^^^^^

.. code-block:: yaml

  resources:
    cluster:
      type: OS::Magnum::Cluster
      properties:
        # required
        cluster_template: String # Name or ID of cluster template
        # optional
        create_timeout: Integer # Timeout for creating cluster in minutes. Defaults to 60 minutes
        discovery_url: String # Specifies a custom discovery url for node discovery
        keypair: String # name of keypair. Uses keypair in template if keypair is not defined here
        master_count: Integer # Number of master nodes, defaults to 1. Must be at least 1
        name: String # name of cluster
        node_count: Integer # Number of worker nodes, defaults to 1. Must be at least 1

  outputs:

    api_address:
      description: Endpoint URL of COE API exposed to end-users
      value: {get_attr: [cluster, api_address]}
    cluster_template_id:
      description: UUID of cluster template
      value: {get_attr: [cluster, cluster_template_id]}
    coe_version:
      description: Version information of container engine in chosen COE in cluster
      value: {get_attr: [cluster, coe_version]}
    create_timeout:
      description: Timeout in minutes for cluster creation
      value: {get_attr: [cluster, create_timeout]}
    discovery_url:
      description: Custom discovery url for node discovery
      value: {get_attr: [cluster, discovery_url]}
    keypair:
      description: Name of keypair
      value: {get_attr: [cluster, keypair]}
    master_addresses:
      description: List of IPs for all master nodes
      value: {get_attrL [cluster, master_addresses]}
    master_count:
      descripition: Number of servers that serve as master for the cluster
      value: {get_attr: [cluster, master_count]}
    name:
      description: Name of the resource
      value: {get_attr:[cluster, name]}
    node_addresses: IP addresses of all worker nodes in the cluster
      description:
      value: {get_attr: [cluster, node_addresses]}
    node_count:
      description: Number of servers that will serve as node for the cluster
      value: {get_attr: [cluster, node_count]}
    show:
      description: Show detailed information about the cluster
      value: {get_attr: [cluster, show]}
    stack_id:
      description: UUID of orchestration stack for this COE cluster
      value: {get_attr: [cluster, stack_id]}
    status:
      description: Status of this COE cluster
      value: {get_attr: cluster, status}
      status_reason:
      description: The reason for the cluster current status
      value: {get_attr: cluster, status_reason}


Example Template
^^^^^^^^^^^^^^^^^
For example, we could have the template ``example.yaml`` which outlines the template for a Kubernetes cluster and instructs heat to create a cluster using this template:

.. code-block:: yaml

  heat_template_version: 2018-08-31 #Train release

  description: This is an example template to create a Kubernetes cluster.

  parameters:
    keypair:
      type: string
      default: mykeypair

    image:
      type: string
      default: <IMAGE_ID>
      description: fedora-atomic

  resources:
    stack_cluster_template:
      type: OS::Magnum::ClusterTemplate
      properties:
        coe: kubernetes
        dns_nameserver: 8.8.8.8
        docker_storage_driver: devicemapper
        docker_volume_size: 10
        external_network: External
        flavor: c1.medium
        floating_ip_enabled: false
        image: {get_param: image}
        labels: {'kube_tag': 'v1.14.3', 'kube_dashboard_enabled': '1', 'heat_container_agent_tag': 'train-stable-3', 'auto_healing': 'true', 'ingress_controller': 'traefik'}
        master_flavor: c1.medium
        name: my-cluster-template
        network_driver: flannel
        registry_enabled: false
        server_type: vm
        volume_driver: cinder
        master_lb_enabled: false

    test_cluster:
      type: OS::Magnum::Cluster
      properties:
        cluster_template: {get_resource: stack_cluster_template}
        create_timeout: 60
        keypair: {get_param: keypair}
        name: test-cluster
        node_count: 1
        master_count: 1


Then we can launch this stack using:

.. code-block:: bash

  openstack stack create -t example.yaml test-cluster-stack


To delete a cluster created using example.yaml, delete the stack which was built by `example.yaml`:

.. code-block:: bash

  openstack stack delete test-cluster-stack

Accessing the Cluster
^^^^^^^^^^^^^^^^^^^^^
To access the cluster, add a floating IP to the master node and ssh using:

.. code-block:: bash

  #For Fedora Atomic
  ssh -i <private-key> fedora@<master-ip>

  #For coreOS
  ssh -i <private-key> core@<master-ip>


Upgrading Clusters
-------------------

Rolling upgrades can be applied to Kubernetes Clusters using the command ``openstack coe cluster upgrade <cluster-id> <new-template-id>``.
This command can be used for upgrading the Kubernetes version or for upgrading the node operating system version.

    **Note:** Downgrading is not supported


openstack coe cluster upgrade
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: bash

  openstack coe cluster upgrade --help
  usage: openstack coe cluster upgrade [-h] [--max-batch-size <max_batch_size>] [--nodegroup <nodegroup>] <cluster> cluster_template

  Upgrade a Cluster

  positional arguments:
    <cluster>             The name or UUID of cluster to update
    cluster_template      The new cluster template ID will be upgraded to.

  optional arguments:
    -h, --help            show this help message and exit
    --max-batch-size <max_batch_size>
                          The max batch size for upgrading each time.
    --nodegroup <nodegroup>
                          The name or UUID of the nodegroup of current cluster.


Example
^^^^^^^^^

This example will go through how to upgrade an existing cluster to use Kubernetes v1.15.7.

The cluster we will update has the following features:

.. code-block:: text

  +----------------------+------------------------------------------------------------------------------------------------------------+
  | Field                | Value                                                                                                      |
  +----------------------+------------------------------------------------------------------------------------------------------------+
  | status               | UPDATE_COMPLETE                                                                                            |
  | health_status        | UNKNOWN                                                                                                    |
  | cluster_template_id  | e186c6e2-dd47-4df0-ac3f-3eb46e64cb3d                                                                       |
  | node_addresses       | ['10.0.0.131', '10.0.0.8']                                                                                 |
  | uuid                 | 686f9fa1-eb56-4c23-9afd-67a79c283736                                                                       |
  | stack_id             | 80b6af23-8a14-4a44-bc62-b77d9eb6736b                                                                       |
  | status_reason        | None                                                                                                       |
  | created_at           | 2020-11-16T12:46:28+00:00                                                                                  |
  | updated_at           | 2020-11-27T11:47:32+00:00                                                                                  |
  | coe_version          | v1.15.7                                                                                                    |
  | labels               | {'auto_healing_controller': 'draino', 'max_node_count': '4', 'kube_tag': 'v1.14.3', 'min_node_count': '1', |
  |                      | 'ingress_controller': 'traefik', 'auto_healing_enabled': 'true', 'heat_container_agent_tag': 'train-       |
  |                      | stable-3', 'auto_scaling_enabled': 'true'}                                                                 |
  | labels_overridden    |                                                                                                            |
  | labels_skipped       |                                                                                                            |
  | labels_added         |                                                                                                            |
  | fixed_network        | None                                                                                                       |
  | fixed_subnet         | None                                                                                                       |
  | floating_ip_enabled  | False                                                                                                      |
  | faults               |                                                                                                            |
  | keypair              | mykeypair                                                                                                  |
  | api_address          | https://10.0.0.117:6443                                                                                    |
  | master_addresses     | ['10.0.0.201']                                                                                             |
  | master_lb_enabled    |                                                                                                            |
  | create_timeout       | 60                                                                                                         |
  | node_count           | 2                                                                                                          |
  | discovery_url        | https://discovery.etcd.io/6b47ff194fc4dcefb3a7d430d69e761c                                                 |
  | master_count         | 1                                                                                                          |
  | container_version    | 1.12.6                                                                                                     |
  | name                 | k8s-cluster                                                                                                |
  | master_flavor_id     | c1.medium                                                                                                  |
  | flavor_id            | c1.medium                                                                                                  |
  | health_status_reason | {'api': 'The cluster k8s-cluster is not accessible.'}                                                      |
  | project_id           | PROJECT_ID                                                                                                 |
  +----------------------+------------------------------------------------------------------------------------------------------------+

To upgrade the Kubernetes version for our cluster, we create a new template where we change the value of the label ``kube_tag`` from v1.14.3 to v1.15.7

.. code-block:: bash

  openstack coe cluster template create --coe kubernetes \
                                        --image cf37f7d0-1d6b-4aab-a23b-df58542c59cb \
                                        --external-network External \
                                        --network-driver flannel \
                                        --volume-driver cinder \
                                        --dns-nameserver 8.8.8.8 \
                                        --flavor c1.medium \
                                        --master-flavor c1.medium \
                                        --docker-volume-size 10 \
                                        --docker-storage-driver devicemapper \
                                        --labels auto_healing_controller=draino,auto_healing_enabled=true,heat_container_agent_tag=train-stable-3,ingress_controller=traefik \
                                        --labels auto_scaling_enabled=true,min_node_count=1,max_node_count=4,kube_tag=1.15.7
                                        --server_type vm
                                        update-template


Then we apply the cluster upgrade to this cluster:

.. code-block:: bash

  openstack coe cluster upgrade 686f9fa1-eb56-4c23-9afd-67a79c283736 <update-template-id>
  # If the command is successful, the following message should be returned:

  Request to upgrade cluster 686f9fa1-eb56-4c23-9afd-67a79c283736 has been accepted.

The cluster will then move into ``UPDATE_IN_PROGRESS`` state while the cluster updates the Kubernetes version. The cluster will move to ``UPDATE_COMPLETE`` status when the upgrade is complete.
We can verify that our cluster is using a different version of Kubernetes by using SSH to connect to the master node and running the following command:


.. code-block:: bash

  $ kubectl version

  Client Version: version.Info{Major:"1", Minor:"15", GitVersion:"v1.15.7", GitCommit:"6c143d35bb11d74970e7bc0b6c45b6bfdffc0bd4", GitTreeState:"clean", BuildDate:"2019-12-11T12:42:56Z", GoVersion:"go1.12.12", Compiler:"gc", Platform:"linux/amd64"}
  Server Version: version.Info{Major:"1", Minor:"15", GitVersion:"v1.15.7", GitCommit:"6c143d35bb11d74970e7bc0b6c45b6bfdffc0bd4", GitTreeState:"clean", BuildDate:"2019-12-11T12:34:17Z", GoVersion:"go1.12.12", Compiler:"gc", Platform:"linux/amd64"}

  $ sudo docker version

  Client:
   Version:         1.13.1
   API version:     1.26
   Package version: docker-1.13.1-68.git47e2230.fc29.x86_64
   Go version:      go1.11.12
   Git commit:      47e2230/1.13.1
   Built:           Sat Aug 17 20:18:33 2019
   OS/Arch:         linux/amd64

  Server:
   Version:         1.13.1
   API version:     1.26 (minimum version 1.12)
   Package version: docker-1.13.1-68.git47e2230.fc29.x86_64
   Go version:      go1.11.12
   Git commit:      47e2230/1.13.1
   Built:           Sat Aug 17 20:18:33 2019
   OS/Arch:         linux/amd64
   Experimental:    false

We can see that the Kubernetes and Docker version have been upgraded for our cluster.


Updating Clusters
------------------

Clusters can be modified using the command:

.. code-block:: bash

  openstack coe cluster update [-h] [--rollback] <cluster> <op> <path=value> [<path=value> ...]

  Update a Cluster

  positional arguments:
    <cluster>     The name or UUID of cluster to update
    <op>          Operations: one of 'add', 'replace' or 'remove'
    <path=value>  Attributes to add/replace or remove (only PATH is necessary on remove)

  optional arguments:
    -h, --help    show this help message and exit
    --rollback    Rollback cluster on update failure.


The following table summarizes the possible changes that can be applied to the cluster.

+---------------+-----+------------------+-----------------------+
| Attribute     | add | replace          | remove                |
+===============+=====+==================+=======================+
| node_count    | no  | add/remove nodes | reset to default of 1 |
+---------------+-----+------------------+-----------------------+
| master_count  | no  | no               | no                    |
+---------------+-----+------------------+-----------------------+
| name          | no  | no               | no                    |
+---------------+-----+------------------+-----------------------+
| discovery_url | no  | no               | no                    |
+---------------+-----+------------------+-----------------------+


Resize a Cluster
-----------------

The size of a cluster can be changed by using the following command:

.. code-block:: bash

  openstack coe cluster resize [-h] [--nodes-to-remove <Server UUID>] [--nodegroup <nodegroup>] <cluster> node_count

  Resize a Cluster

  positional arguments:
    <cluster>             The name or UUID of cluster to update
    node_count            Desired node count of the cluser.

  optional arguments:
    -h, --help            show this help message and exit
    --nodes-to-remove <Server UUID>
                          Server ID of the nodes to be removed. Repeat to addmore server ID
    --nodegroup <nodegroup>
                          The name or UUID of the nodegroup of current cluster.




References:
-----------

https://docs.openstack.org/magnum/train/user/

https://docs.openstack.org/heat/train/template_guide/openstack.html

https://www.openstack.org/videos/summits/austin-2016/intro-to-openstack-magnum-with-kubernetes

https://object-storage-ca-ymq-1.vexxhost.net/swift/v1/6e4619c416ff4bd19e1c087f27a43eea/www-assets-prod/presentation-media/openstack-magnum-hands-on.pdf

https://www.openstack.org/videos/summits/denver-2019/container-use-cases-and-developments-at-the-cern-cloud
