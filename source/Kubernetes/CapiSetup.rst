#####################################
Setup a new cluster using Cluster API
#####################################

.. warning:: Under Testing
    Cluster API is currently under release testing
    between interested users and the cloud team.
    It is not rated for production workloads at this time.

.. contents::

Deployment Considerations
=========================

Management Machine
------------------

For production workloads, it's recommended to create a dedicated cluster management machine. This should
only be accessable to Cluster Administrators and holds a copy of the kubeconfig. The kubeconfig cannot
be recovered (unlike Magnum) after generation. Additionally, a known working dedicated machine can be used
to quickly gain access and perform recovery or upgrades as required.

Account Security
----------------

A ``clouds.yaml`` file with the application credentials is also required. The credentials should not be unrestricted,
as a compromised cluster would allow an attacker to create additional credentials.
This file should be removed or restricted on shared machines to prevent unauthorized access.

Preparing to Deploy
===================

Background
----------

A Kubernetes cluster is required to create the cluster. To break this "chicken-egg" problem
a minikube cluster is created to bootstrap the main cluster. It is not
recommended to use this cluster for any production workloads.

Bootstrap Machine Prep
----------------------

A Ubuntu machine is used to provide the minikube cluster. This should use the normal
cloud Ubuntu image, not the stripped down CAPI image designed for nodes.

The following packages are required and
can be installed and configured using the following commands:

- Docker is used to run a Kubernetes staging cluster locally

.. code-block:: bash

    # Docker
    sudo apt install -y docker.io
    sudo usermod -aG docker $USER

- You will need to exit and login again if you have added yourself to the docker group (usermod).
  This is to pick up the new group membership
- Snap is used to install kubectl and Helm

.. code-block:: bash

    sudo apt-get update && sudo apt-get install -y snapd
    export PATH=$PATH:/snap/bin
    sudo snap install kubectl --classic
    sudo snap install helm --classic

- Install minikube and start a cluster:

.. code-block:: bash

    wget https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
    chmod +x minikube-linux-amd64
    sudo mv minikube-linux-amd64 /usr/local/bin/minikube
    minikube start --driver=docker

- Install clusterctl and the Openstack provider into your minikube cluster:

.. code-block:: bash

    # YQ (Used by Cluster API)
    sudo snap install yq
    # Clusterctl
    curl -L https://github.com/kubernetes-sigs/cluster-api/releases/download/v1.3.3/clusterctl-linux-amd64 -o clusterctl
    chmod +x ./clusterctl
    sudo mv ./clusterctl /usr/local/bin/clusterctl
    clusterctl init --infrastructure openstack

If you run into GitHub rate limiting you will have to generate a personal API token as described
`here. <https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/creating-a-personal-access-token>`_
This only requires the Repo scope, and is set on the CLI as follows

.. code-block:: bash

    export GITHUB_TOKEN=<your token>


These setup steps only have to be completed once per management machine.

Openstack Preparation
---------------------

- `Ensure a dedicated floating IP exists. <https://openstack.stfc.ac.uk/project/floating_ips/>`_ If required, allocate an IP to the project from the External pool.
- Clone https://github.com/stfc/cloud-capi-values

clouds.yaml Prep
----------------

- Generate your application credentials: :ref:`Openstack Application Credentials<application_credentials>`. 
  It is recommended you use Horizon (the web interface) to download the clouds.yaml file.

The ``clouds.yaml`` file should have the following format:

.. code-block:: yaml

    clouds:
        openstack:
            auth:
            auth_url: https://openstack.stfc.ac.uk:5000/v3
            application_credential_id: ""
            application_credential_secret: ""
        region_name: "RegionOne"
        interface: "public"
        identity_api_version: 3
        auth_type: "v3applicationcredential"

- Add the UUID of the project you want to create the cluster in. This is the project ID under the Openstack section which is omitted by default. 
  This can be found `here <https://openstack.stfc.ac.uk/identity/>`_.

Your ``clouds.yaml`` should now look like:

.. code-block:: yaml

    clouds:
        openstack:
          auth:
          auth_url: https://openstack.stfc.ac.uk:5000/v3
          application_credential_id: ""
          application_credential_secret: ""
          project_id: ""
        region_name: "RegionOne"
        interface: "public"
        identity_api_version: 3
        auth_type: "v3applicationcredential"

- Place this file in the ``cloud-capi-values`` directory you cloned earlier.

Creating the cluster
====================

Yaml Files
----------

The configuration is spread across multiple yaml files to make it easier to manage.
These are as follows:

- ``values.yaml`` contains the default values for the cluster using the STFC Cloud service. These should not be changed.

-  ``user-values.yaml`` contains some values that must be set by the user. There are also optional values that can be changed too for advanced users.

- ``flavors.yaml`` contains the Openstack flavors to use for worker nodes. Common flavors are provided and can be uncommented and changed as required. By default the cluster will use ``l3.nano`` workers by default if unspecified.

- ``clouds.yaml`` contains the Openstack application credentials. This file should be in the same directory as the other yaml files.

The cloud team will periodically update ``flavors.yaml``, ``values.yaml``, and ``user-values.yaml`` to reflect changes in the STFC Cloud service.
These include new versions of Kubernetes or machine images, best practices, new flavors...etc. A user will pull these changes 
by running ``git pull`` in the ``cloud-capi-values`` directory in the future.


Configuring the cluster
-----------------------

- The mandatory values in ``user-values.yaml`` must be set. Optional
  values may also be changed as required.

- The ``flavors.yaml`` file contains the Openstack flavors to use for worker nodes. 
  These can be changed as required but will use ``l3.nano`` by default if unspecified.

.. code-block:: bash

    cd cloud-capi-values
    export CLUSTER_NAME="demo-cluster"  # or your cluster name
    
    # Install the custom resource definitions (CRDs)
    helm repo add capi-addons https://stackhpc.github.io/cluster-api-addon-provider
    helm upgrade cluster-api-addon-provider capi-addons/cluster-api-addon-provider --install --version ">=0.1.0-dev.0.main.0,<0.1.0-dev.0.main.9999999999" --wait

    # Deploy the cluster called "demo-cluster"
    helm repo add capi https://stackhpc.github.io/capi-helm-charts
    helm upgrade $CLUSTER_NAME capi/openstack-cluster --install -f values.yaml -f clouds.yaml -f user-values.yaml -f flavors.yaml --wait

- Progress can be monitored with the following command in a separate terminal:

.. code-block:: bash

    kubectl logs deploy/capo-controller-manager -n capo-system -f

- When the deployment is complete clusterctl will report the cluster as Ready: True

.. code-block:: bash

    clusterctl describe cluster $CLUSTER_NAME

Moving the control plane
========================

At this point the control plane is still on the minikube cluster. This is not recommended for
long-lived or production workloads. We can pivot the cluster to self-manage:

.. warning::

    After moving the control plane the kubeconfig cannot be retrieved if lost.
    Ensure a copy of the kubeconfig is placed into secure storage for production clusters.

Moving from minikube cluster
----------------------------

- Install clusterctl into the new cluster and move the control plane

.. code-block:: bash

    clusterctl get kubeconfig $CLUSTER_NAME > kubeconfig.$CLUSTER_NAME
    clusterctl init --infrastructure openstack --kubeconfig=kubeconfig.$CLUSTER_NAME
    clusterctl move --to-kubeconfig kubeconfig.$CLUSTER_NAME

- Ensure the control plane is now running on the new cluster:

.. code-block:: bash

    kubectl get kubeadmcontrolplane --kubeconfig=kubeconfig.$CLUSTER_NAME

minikube Shutdown
-----------------

- Replace the existing minikube kubeconfig with the new cluster's kubeconfig

.. code-block:: bash

    cp -v kubeconfig.$CLUSTER_NAME ~/.kube/config
    # Ensure kubectl now uses the new kubeconfig displayed the correct nodes:
    kubectl get nodes
    
    # Update the cluster to ensure everything lines up with your helm chart
    helm upgrade cluster-api-addon-provider capi-addons/cluster-api-addon-provider --install --version ">=0.1.0-dev.0.main.0,<0.1.0-dev.0.main.9999999999" --wait
    helm upgrade $CLUSTER_NAME capi/openstack-cluster --install -f values.yaml -f clouds.yaml -f user-values.yaml -f flavors.yaml --wait

    # Check the cluster status
    clusterctl describe cluster $CLUSTER_NAME

- Remove minikube bootstrap cluster

.. code-block:: bash

    minikube delete

Your cluster is now complete
