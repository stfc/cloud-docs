===========================
Create a Kubernetes Cluster
===========================

.. warning::  Deprecated

  Magnum is deprecated and will be replaced in the future with Cluster API. 

Clusters are groups of resources (nova instances, neutron networks, security groups etc.) combined to function as one system.
To do this, Magnum uses Heat to orchestrate and create a stack which contains the cluster.

This documentation will focus on how to create Kubernetes clusters using OpenStack Magnum.

Magnum CLI
##########

Any commands for creating clusters using OpenStack Magnum begin with:

.. code-block:: bash

  openstack coe <commands> <options>

In order to have the openstack commands for Magnum available to use through the CLI, you will need to install the
python client for Magnum. This can be done using pip:

.. code-block:: bash

  pip install python-magnumclient

Now the commands relating to the container orchestration engine, clusters, cluster templates are available on the
command line.


Cluster Templates
#################

Clusters can be created from templates which are passed through Heat. To view the
list of cluster templates which are in your project, you can use the following command:

.. code-block:: bash

  openstack coe cluster template list

  # This should return either an empty line if there are no templates, or
  # a table similar to the one below:
  +--------------------------------------+------------------------------+
  | uuid                                 | name                         |
  +--------------------------------------+------------------------------+
  | e186c6e2-dd47-4df0-ac3f-3eb46e64cb3d | kubernetes-v1_14_3           |
  | 0bd1232d-06f2-42ca-b6d5-c27e57f26c3c | kubernetes-ha-master-v1_14_3 |
  | a07903d0-aecf-4f15-a35f-f4fd74060e2f | coreos-kubernetes-v1_14_3    |
  +--------------------------------------+------------------------------+

Templates can be created using the following command:

.. code-block:: text

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


Kubernetes Cluster Template:

.. code-block:: bash

  $ openstack coe cluster template show e186c6e2-dd47-4df0-ac3f-3eb46e64cb3d
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



Create a Kubernetes Cluster
###########################

We can create a Kubernetes cluster using one of the cluster templates that are available. To create a cluster, we use the command:

.. code-block:: bash

  openstack coe cluster create  --cluster-template <cluster-template> --discovery-url <discovery-url> --master-count <master-count> --node-count <node-count>
                                --timeout <timeout> --merge-labels
                                #The following options can be used to overwrite the same options in the cluster template
                                --docker-volume-size <docker-volume-size>
                                --labels <KEY1=VALUE1,KEY2=VALUE2;KEY3=VALUE3...> # --merge-labels flag will add labels to the ones provided by the template
                                --keypair <keypair>
                                --master-flavor <master-flavor>
                                --flavor <flavor>
                                --fixed-network <fixed-network>
                                --fixed-subnet <fixed-subnet>
                                --floating-ip-enabled
                                --floating-ip-disabled
                                --master-lb-enabled
                                <name>


For example, consider a user that wants to create a cluster using the Kubernetes cluster template. They want the cluster to have:
  - one master node
  - one worker node
  - their keypair mykeypair

.. code-block:: bash

  openstack coe cluster create --cluster-template e186c6e2-dd47-4df0-ac3f-3eb46e64cb3d \
                               --keypair mykeypair \
                               --master-count 1 \
                               --node-count 1 \
                               kubernetes-cluster-test-1

  #This should return an output similar to this one
  Request to create cluster 27cdcad8-375f-4d4f-a186-8fa99b80c5c5 accepted
  #This indicates that the command was successful and the cluster is being built

A cluster containing one master node and one worker node takes approximately 14 minutes to build.
By default, cluster creation times out at 60 minutes.


After the cluster has been created, you can associate a floating IP to the master node
and SSH into the node using:

```
ssh -i <mykeypair-private-key> fedora@<floating_ip>
```



References
----------

https://docs.openstack.org/magnum/train/user/#

https://clouddocs.web.cern.ch/containers/quickstart.html
