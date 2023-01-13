#######################
ClusterAPI Introduction
#######################

Cluster API is a platform agnostic way of managing physical resources (i.e. VMs) using
Kubernetes.

This provides a nuber of advantages including:
- Familiarity to experience K8s admins, as nodes act similarly to pods...etc.
- Tooling which directly supports Openstack, including resource creation and management.
- Support and fixes for newer Kuberenetes versions without waiting for Openstack upgrades.

The Kubernetes image provided is based on 
`upstream's Ubuntu image <https://github.com/kubernetes-sigs/image-builder/tree/master/images/capi>`_,
which has been forked to comply with UKRI security policy `here <https://github.com/stfc/k8s-image-builder>`_.

Unlike Magnum, the Kubernetes version supported is completely decoupled from Openstack, with upstream
typically supporting the N-1 major version (where N is the latest release).

