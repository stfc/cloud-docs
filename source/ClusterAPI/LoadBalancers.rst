=====================================
Using Openstack Load Balancers in K8s
=====================================

Background
==========

Load balancers are managed by the external Openstack controller
driver. This is deployment agnostic, i.e. can be used on Cluster API,
RKE and other K8s deployments.

Setup
=====

- Follow the steps to setup the clouds file: :ref:`clouds_yaml` 
- Clone the ClusterAPI Openstack scripts:

.. code-block:: bash

    git clone https://github.com/kubernetes-sigs/cluster-api-provider-openstack
    cd cluster-api-provider-openstack

- Generate the secret containing the clouds conf:

.. code-block:: bash

    # Substitute <cloud> with the name from the clouds.yaml file
    templates/create_cloud_conf.sh ~/.config/openstack/clouds.yaml <cloud> > /tmp/cloud.conf
    kubectl create secret -n kube-system generic cloud-config --from-file=/tmp/cloud.conf
    rm /tmp/cloud.conf

- Deploy the out of tree controller module

.. code-block:: bash

    kubectl apply -f https://raw.githubusercontent.com/kubernetes/cloud-provider-openstack/master/manifests/controller-manager/cloud-controller-manager-roles.yaml
    kubectl apply -f https://raw.githubusercontent.com/kubernetes/cloud-provider-openstack/master/manifests/controller-manager/cloud-controller-manager-role-bindings.yaml
    kubectl apply -f https://raw.githubusercontent.com/kubernetes/cloud-provider-openstack/master/manifests/controller-manager/openstack-cloud-controller-manager-ds.yaml

- Check the `Openstack controller manager` deploys:

.. code-block:: bash

    kubectl get pods -n kube-system

Usage
=====

The Openstack Controller Manager will automatically register
itself as a loadbalancer provider. 

Deployments / charts which use the `loadbalancer` port type 
will automatically provision a load balancer within Openstack.
Additionally, a floating IP can be specified (this must belong
to the Openstack project) and will be used for an idempotent deployment.