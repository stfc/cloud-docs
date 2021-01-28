=========================
JupyterHub on Kubernetes
=========================

    **Important:** Magnum is currently in technical preview on STFC Cloud. If you have any feedback or suggestions please send it to cloud-support@gridpp.rl.ac.uk

This documentation assumes that you will be installing JupyterHub on a Kubernetes cluster that has been created using OpenStack Magnum.

In this tutorial we will break the installation down into the following:
  - Create a cluster template and launch a Kubernetes cluster using OpenStack Magnum
  - Install Helm v3 and define persistent volume for the cluster
  - Install JupyterHub

Creating a Kubernetes Cluster
-----------------------------

The template for the cluster:

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
                                        --labels kube_tag=v1.14.3,kube_dashboard_enabled=1,heat_container_agent_tag=train-stable-3,auto_healing=true,ingress_controller=traefik
                                        --server_type vm
                                        test-template



Create a cluster:

.. code-block:: bash

  openstack coe cluster create --cluster-template test-template \
                               --keypair mykeypair \
                               --docker-volume-size 10  \
                               --master-count 1 \
                               --node-count 1 \
                               test-cluster

    #This should return an output similar to this one
    Request to create cluster 27cdcad8-375f-4d4f-a186-8fa99b80c5c5 accepted
    #This indicates that the command was successful and the cluster is being built

Once the cluster has been created successfully, we can associate a floating IP to the master node VM and then SSH into the cluster:

.. code-block:: bash

  ssh -i mykeypair.key fedroa@FLOATING_IP

  #This should return something similar to:

  Last login: Fri Sep 18 13:17:02 2020 from 130.XXX.XXX.XXX

  [fedora@test-template-vbo5u2doyiao-master-0 ~]$

  #You have now successfully connected to the master node


Configure Storage
~~~~~~~~~~~~~~~~~~

Magnum does not automatically configure cinder storage for clusters.


The storage class can be defined using a YAML file. For example we could define the storage class to be:

**YAML File from: https://github.com/zonca/jupyterhub-deploy-kubernetes-jetstream/blob/master/kubernetes_magnum/storageclass.yaml_**

.. code-block:: yaml

  apiVersion: storage.k8s.io/v1

  kind: StorageClass

  metadata:

    name: standard

    annotations:

      storageclass.beta.kubernetes.io/is-default-class: "true"

    labels:

      kubernetes.io/cluster-service: "true"

      addonmanager.kubernetes.io/mode: EnsureExists

  provisioner: kubernetes.io/cinder


Then we create the storage class:

.. code-block:: bash

  kubectl create -f storageclass.yaml


Helm v3
--------

The Train release supports Helm v2 charts being installed and supports labels for installing Tiller.

However, it is possible to install and run charts for Helm v3.

    In the Ussuri release onwards, Magnum supports the use of a label to install Helm v3 client. This label can be added to a template or at cluster creation time.

**Note:** Helm v2 reaches end of support in November 2020


To install Helm 3:


.. code-block:: bash

  curl https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3 | bash


Other methods for installing Helm v3 can be found here: https://helm.sh/docs/intro/install/

Now Helm v3 has been installed, we can install JupyterHub.


JupyterHub
-----------

The following is the tutorial from the **_Zero to JupyterHub with Kubernetes_** installation documentation.

.. code-block:: bash

  # Generate a random hex string
  openssl rand -hex 32  #copy the output

Then create a file called config.yaml and write the following:

.. code-block:: bash

  vi config.yaml # fedora doesn't use nano


.. code-block:: yaml

  proxy:
    secretToken: "<RANDOM_HEX>" #this is the random string which you have copied


Next is to add the JupyterHub Helm chart to your chart repository and install it.

.. code-block:: bash

  helm repo add jupyterhub https://jupyterhub.github.io/helm-chart/
  helm repo update

  RELEASE=jhub
  NAMESPACE=jhub

  helm upgrade --cleanup-on-fail \
    --install $RELEASE jupyterhub/jupyterhub \
    --namespace $NAMESPACE \
    --create-namespace \
    --version=0.9.0 \
    --values config.yaml \
    --timeout 30m0s #This is to stop the installation from timing out


When installation is complete it should return a message similar to the following:

.. code-block:: txt

  NAME: jhub
  LAST DEPLOYED: Tue Oct 13 11:01:15 2020
  NAMESPACE: jhub
  STATUS: deployed
  REVISION: 1
  TEST SUITE: None
  NOTES:
  Thank you for installing JupyterHub!

  Your release is named jhub and installed into the namespace jhub.

  You can find if the hub and proxy is ready by doing:

   kubectl --namespace=jhub get pod

  and watching for both those pods to be in status 'Running'.

  You can find the public IP of the JupyterHub by doing:

   kubectl --namespace=jhub get svc proxy-public

  It might take a few minutes for it to appear!

  Note that this is still an alpha release! If you have questions, feel free to
    1. Read the guide at https://z2jh.jupyter.org
    2. Chat with us at https://gitter.im/jupyterhub/jupyterhub
    3. File issues at https://github.com/jupyterhub/zero-to-jupyterhub-k8s/issues


References:
-----------

https://github.com/zonca/jupyterhub-deploy-kubernetes-jetstream

https://zonca.dev/2020/05/kubernetes-jupyterhub-jetstream-magnum.html

https://zero-to-jupyterhub.readthedocs.io/en/latest/

https://helm.sh/docs/intro/install/
