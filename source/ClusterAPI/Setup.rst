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

Deployment Account
------------------

The deployment is tied to the account which created it. This includes their
credentials being encoded as an opaque secret within the cluster for the cluster API openstack component.

For production workloads, and long-lived clusters with multiple accounts a service account it recommended.
This means the cluster isn't tied to the lifetime of a single user's account or password. A service
account can be requested through our :ref:`contact_details`.

Management Machine
------------------

For production workloads, it's recommended to create a dedicated cluster management machine. This should
only be accessable to Cluster Administrators and holds a copy of the kubeconfig. The kubeconfig cannot
be recovered (unlike Magnum) after generation. Additionally, a known working dedicated machine can be used
to quickly gain access and perform recovery or upgrades as required.

Account Security
----------------

A `clouds.yaml` file with the user's password is also required. This file should be removed or restricted
on shared machines to prevent unauthorized access.

Preparing to Deploy
===================

Background
----------

A Kuberenetes cluster is required to create the cluster. To break this "chicken-egg" problem
a Kuberenetes-in-Docker (KinD) cluster is created to bootstrap the main cluster. It is not
recommended to use this cluster for any production workloads.

Bootstrap Machine Prep
----------------------

A Ubuntu machine is used to provide the KinD cluster. This should use the normal
cloud Ubuntu image, not the stripped down CAPI image designed for nodes.

The following packages are required and
can be installed and configured using the following commands:

.. code-block:: bash

    # Docker
    sudo apt install -y docker.io
    sudo usermod -aG docker $USER
    # Snap
    sudo apt-get update && sudo apt-get install -y snapd
    export PATH=$PATH:/snap/bin
    # Kubectl
    sudo snap install kubectl --classic
    # KinD
    sudo snap install go --classic
    export PATH="/home/$USER/go/bin:$PATH"
    go install sigs.k8s.io/kind@v0.14.0
    # YQ (Used by Cluster API)
    sudo snap install yq
    # Clusterctl
    curl -L https://github.com/kubernetes-sigs/cluster-api/releases/download/v1.2.4/clusterctl-linux-amd64 -o clusterctl
    chmod +x ./clusterctl
    sudo mv ./clusterctl /usr/local/bin/clusterctl

- You will need to exit and login again if you have installed docker to pick up the new group membership
- Go modules can be permanently added to the path by appending the following to the user's .bashrc file:
  Then source ~/.bashrc or logout and login again.

.. code-block:: bash

    export PATH="/home/$USER/go/bin:$PATH"


clouds.yaml Prep
----------------

- Follow the steps in: :ref:`clouds_yaml`
- Currently clusterctl will attempt to verify the CA chain using a provided public CA certificate, but this is not required for Openstack components.
  `verify: false` **must** be added to clouds.yaml:

The resulting file should look like:

.. code-block:: yaml

    clouds:
        openstack:
            auth:
                auth_url: https://openstack.stfc.ac.uk:5000/v3
                username: "username"
                password: "password"
                project_id: project_id
                project_name: "project_name"
                user_domain_name: "stfc"
            region_name: "RegionOne"
            verify: false
            interface: "public"
            identity_api_version: 3 

*Note: Keypairs are associated with individual accounts. You may need to
create a new keypair if you are using a service account.*

Openstack Preparation
---------------------

- `Ensure a keypair exists <https://openstack.stfc.ac.uk/project/key_pairs>`_
- `Make a note of the latest public Cluster API image name and K8s version <https://openstack.stfc.ac.uk/project/images>`_
- `Ensure a dedicated floating IP exists. <https://openstack.stfc.ac.uk/project/floating_ips/>`_ If required, allocate an IP to the project from the External pool.

Creating the cluster
====================

Configuring the cluster
-----------------------

- Prepare the environment variables with details from the pre-prepared Openstack file:

.. code-block:: bash

    wget https://raw.githubusercontent.com/kubernetes-sigs/cluster-api-provider-openstack/master/templates/env.rc -O /tmp/env.rc
    # Substitute openstack for the name in your clouds.yaml, typically "openstack"
    source /tmp/env.rc ~/.config/openstack/clouds.yaml openstack
    export OPENSTACK_DNS_NAMESERVERS=130.246.209.132
    export OPENSTACK_FAILURE_DOMAIN=ceph
    export OPENSTACK_EXTERNAL_NETWORK_ID=External

- The following environment variables should be set based on user requirements.
  (If you need GPU or other specialised nodes it's recommended to use a generic VM such as l2.tiny then
  create a machine deployment as described in TODO)

.. code-block:: bash

    export OPENSTACK_CONTROL_PLANE_MACHINE_FLAVOR=<flavour>
    export OPENSTACK_NODE_MACHINE_FLAVOR=<flavour>
    # The public cluster API image as found in the Openstack web interface
    export OPENSTACK_IMAGE_NAME=<image_name>
    # The SSH key pair name from in the Openstack web interface
    export OPENSTACK_SSH_KEY_NAME="<ssh key pair name>"

- Create the KinD bootstrap cluster:

.. code-block:: bash

    kind create cluster && kubectl cluster-info

- Pick a name for the cluster, this will be used in subsequent commands:

.. code-block:: bash

    export CLUSTER_NAME=demo

- Initialise clusterctl on the KinD bootstrap cluster

.. code-block:: bash

    clusterctl init --infrastructure openstack

- Generate the cluster config:

.. code-block:: bash

    # This is based on the K8s in the built image
    # It's recommended to have an odd quorum of control machines, i.e. 1/3/5
    clusterctl generate cluster $CLUSTER_NAME \
    --kubernetes-version v1.x.y \
    --control-plane-machine-count=3 \
    --worker-machine-count=1 > $CLUSTER_NAME.yaml

- Edit the generated `$CLUSTER_NAME.yaml` file to specify the allocated floating IP. 

.. warning::

    If the selected floating IP is already being used by an existing load balancer in the
    same project it will be disassociated and re-allocated to the new load balancer.

The existing block needs changing from:

.. code-block:: yaml

    spec:
      apiServerLoadBalancer:
        enabled: true

To the floating IP pre-allocated in the project:

.. code-block:: yaml

    spec:
      apiServerFloatingIP: 130.246.x.y
      apiServerLoadBalancer:
        enabled: true

This ensures the cluster load balancer will always use the same address, and will not
use the entire project's quota allocating new floating IPs if there are any problems.

Provisioning the new cluster
----------------------------

- Create the cluster by applying the generated cluster definition:

.. code-block:: bash

    kubectl apply -f $CLUSTER_NAME.yaml

- Openstack deployment can be optionally monitored with

.. code-block:: bash

    kubectl logs deploy/capo-controller-manager -n capo-system -f

- Wait for `kubectl get kubeadmcontrolplane` to show the control plane initialised but unavailable:

.. code-block::

    NAME                    CLUSTER   INITIALIZED   API SERVER AVAILABLE   REPLICAS   READY   UPDATED   UNAVAILABLE   AGE     VERSION
    demo-control-plane      demo-v1   true                                 2                  2         2             6m47s   v1.x.y

- Download the kubeconfig for the new cluster:

.. code-block:: bash

    clusterctl get kubeconfig $CLUSTER_NAME > $CLUSTER_NAME.kubeconfig

- Deploy a networking overlay. This tutorial assumes the use of Calico. The latest release can be found `here <https://projectcalico.docs.tigera.io/release-notes/>`_

.. code-block:: bash

    kubectl --kubeconfig=$CLUSTER_NAME.kubeconfig apply -f https://docs.projectcalico.org/manifests/calico.yaml

- The remaining nodes will now come up and show as ready in `kubectl get nodes --kubeconfig $CLUSTER_NAME.kubeconfig`

Moving the control plane
========================

At this point the control plane is still on the KinD cluster. This is not recommended for
long-lived or production workloads. We can pivot the cluster to self-manage:

.. warning::

    After moving the control plane the kubeconfig cannot be retrieved if lost.
    Ensure a copy of the kubeconfig is placed into secure storage for production clusters.

Moving from KinD cluster
------------------------

- Install clusterctl into the new cluster and move the control plane

.. code-block:: bash

    clusterctl init --infrastructure openstack --kubeconfig=$CLUSTER_NAME.kubeconfig
    clusterctl move --to-kubeconfig $CLUSTER_NAME.kubeconfig

- Ensure the control plane is now running on the new cluster:

.. code-block:: bash

    kubectl get kubeadmcontrolplane --kubeconfig=$CLUSTER_NAME.kubeconfig

KinD Shutdown
-------------

- Replace the existing KinD kubeconfig with the new cluster's kubeconfig

.. code-block:: bash

    cp -v $CLUSTER_NAME.kubeconfig ~/.kube/config
    # Ensure kubectl now uses the new kubeconfig displayed the correct nodes:
    kubectl get nodes

- Remove KinD bootstrap cluster

.. code-block:: bash

    kind delete cluster

