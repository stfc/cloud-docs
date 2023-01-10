============================
OpenStack Magnum Users
============================

.. warning::  Deprecated

  Magnum is deprecated and will be replaced in the future with Cluster API. 

When a cluster is created, Magnum creates unique credentials for each cluster. This allows the cluster to make changes to its structure
(e.g. create load balancers for specific services, create and attach cinder volumes, update the stack, etc.) without exposing the user's cloud credentials.


How to find the Magnum User Credentials
#######################################

We can obtain the cluster credentials directly from the VM which the master node is on.
First, SSH into the master node's VM and then:

.. code-block:: bash

  [fedora@cluster-master-0 ~]$ cd /etc
  [fedora@cluster-master-0 /etc]$ cd kubernetes/
  [fedora@cluster-master-0 kubernetes]$ ls
  #This will return the list of items similar to:

  apiserver      cloud-config       controller-manager            kube_openstack_config  manifests              scheduler
  ca-bundle.crt  cloud-config-occm  get_require_kubeconfig.sh     kubelet                proxy
  certs          config             keystone_webhook_config.yaml  kubelet-config.yaml    proxy-kubeconfig.yaml

  #The cluster's credentials can be found in the file 'cloud-config'
  #Print the config to the terminal
  [fedora@cluster-master-0 kubernetes]$ cat cloud-config

This will return the cloud-config file containing the cluster's credentials similar to:

.. code-block:: bash

  [Global]
  auth-url=https://AUTH-URL
  user-id=CLUSTER_USER_ID
  password=PASSWORD
  trust-id=TRUST-ID
  ca-file=/etc/kubernetes/ca-bundle.crt
  region=RegionOne
  [LoadBalancer] #with the octavia ingress controller enabled
  use-octavia=True
  subnet-id=SUBNET_ID
  floating-network-id=FLOATING_NETWORK_ID
  create-monitor=yes
  monitor-delay=1m
  monitor-timeout=30s
  monitor-max-retries=3
  [BlockStorage]
  bs-version=v2


These global variables should be used when setting up configmaps such as magnum-auto-healer configmap. **Never** use your own cloud credentials in Kubernetes configmaps. These
would be visible to anyone who has access to the master node in the cluster.
