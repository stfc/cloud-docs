====================
Autoscaling Clusters
====================

    **Important:** Magnum is currently in technical preview on STFC Cloud. If you have any feedback or suggestions please send it to cloud-support@stfc.ac.uk

The Cluster Autoscaler (CA) is a feature in OpenStack Magnum that can be enabled in order for the cluster to scale up or down the worker nodegroup.
The default version which the Train release uses is v1.0. The version of CA to use can be changed at cluster creation by using the label ``autoscaler_tag``

This feature can be enabled by using the label ``auto_scaling_enabled=true`` in a cluster template or at cluster creation.

  **Note:** The CA will start attempting to scale down nodes as long as the machine IDs for each node matches their VM ID.

Machine IDs
-------------

On nodes in a Kubernetes cluster, the system UUID matches the ID of the VM hosting that node. However, the Cluster Autoscaler uses the machine ID to refer to the node when the cluster needs to be scaled down and a node removed.
Kubernetes reads the ID for the VMs from the file ``/etc/machine-id`` on each VM in the cluster. However, these IDs may not match the IDs of the VMs. If the machine ID and system UUID (VM ID) on a node do not match, then the following errors may be present in the CA pod's log:

.. code-block:: text

  E1215 12:59:58.084833       1 scale_down.go:932] Problem with empty node deletion: failed to delete cluster-node-5: manager error deleting nodes: could not find stack indices for nodes to be deleted: 1 nodes could not be resolved to stack indices
  E1215 12:59:58.084958       1 static_autoscaler.go:430] Failed to scale down: failed to delete at least one empty node: failed to delete cluster-node-5: manager error deleting nodes: could not find stack indices for nodes to be deleted: 1 nodes could not be resolved to stack indices
  I1215 13:10:09.265757       1 scale_down.go:882] Scale-down: removing empty node cluster-node-5
  I1215 13:10:18.362187       1 magnum_manager_heat.go:347] Could not resolve node {Name:cluster-node-5 MachineID:d6580d63b98346daacd54c644f76bbd6 ProviderID:openstack:///d07e9c8f-e7dd-4342-9ba6-f5c912afc04e IPs:[10.0.0.8]} to a stack index


To update the machine ID to match the VM ID, the file can be edited directly using:

.. code-block:: bash

  vi /etc/machine-id

  #Then replace the machine ID with the VM's ID

After a few minutes Kubernetes will have updated the IDs for the nodes on those VMs. The system UUID and the machine ID can be seen using ``kubectl describe node <node-name>``.

For example:

.. code-block:: bash

  $ kubectl describe node cluster-node-3
  Name:               cluster-node-3
  Roles:              <none>
  Labels:             beta.kubernetes.io/arch=amd64
                      beta.kubernetes.io/instance-type=8
                      beta.kubernetes.io/os=linux
                      draino-enabled=true
                      failure-domain.beta.kubernetes.io/region=RegionOne
                      failure-domain.beta.kubernetes.io/zone=ceph
                      kubernetes.io/arch=amd64
                      kubernetes.io/hostname=cluster-node-3
                      kubernetes.io/os=linux
                      magnum.openstack.org/nodegroup=default-worker
                      magnum.openstack.org/role=worker
  Annotations:        flannel.alpha.coreos.com/backend-data: {"VtepMAC":"7e:d2:52:53:d3:8c"}
                      flannel.alpha.coreos.com/backend-type: vxlan
                      flannel.alpha.coreos.com/kube-subnet-manager: true
                      flannel.alpha.coreos.com/public-ip: 10.0.0.131
                      node.alpha.kubernetes.io/ttl: 0
                      volumes.kubernetes.io/controller-managed-attach-detach: true
  CreationTimestamp:  Mon, 16 Nov 2020 15:45:32 +0000
  Taints:             <none>
  Unschedulable:      false
  Conditions:
    Type                 Status  LastHeartbeatTime                 LastTransitionTime                Reason                       Message
    ----                 ------  -----------------                 ------------------                ------                       -------
    KernelDeadlock       False   Tue, 15 Dec 2020 13:06:50 +0000   Fri, 27 Nov 2020 11:45:15 +0000   KernelHasNoDeadlock          kernel has no deadlock
    ReadonlyFilesystem   False   Tue, 15 Dec 2020 13:06:50 +0000   Fri, 27 Nov 2020 11:45:15 +0000   FilesystemIsNotReadOnly      Filesystem is not read-only
    MemoryPressure       False   Tue, 15 Dec 2020 13:07:32 +0000   Mon, 16 Nov 2020 15:45:32 +0000   KubeletHasSufficientMemory   kubelet has sufficient memory available
    DiskPressure         False   Tue, 15 Dec 2020 13:07:32 +0000   Mon, 16 Nov 2020 15:45:32 +0000   KubeletHasNoDiskPressure     kubelet has no disk pressure
    PIDPressure          False   Tue, 15 Dec 2020 13:07:32 +0000   Mon, 16 Nov 2020 15:45:32 +0000   KubeletHasSufficientPID      kubelet has sufficient PID available
    Ready                True    Tue, 15 Dec 2020 13:07:32 +0000   Fri, 27 Nov 2020 11:44:41 +0000   KubeletReady                 kubelet is posting ready status
  Addresses:
    InternalIP:  10.0.0.131
  Capacity:
   cpu:                2
   ephemeral-storage:  39922Mi
   hugepages-1Gi:      0
   hugepages-2Mi:      0
   memory:             4030348Ki
   pods:               110
  Allocatable:
   cpu:                2
   ephemeral-storage:  37675125903
   hugepages-1Gi:      0
   hugepages-2Mi:      0
   memory:             3927948Ki
   pods:               110
  System Info:
   Machine ID:                 54d6093e-0b15-4c92-80f7-5f3126e06083
   System UUID:                54d6093e-0b15-4c92-80f7-5f3126e06083
   Boot ID:                    dddb9b89-559c-4f3f-8b1f-6b6f0d5a62dd
                                                                            ..................


This shows that this node had the machine ID updated so that it now matches the System UUID and will refer to the VM by the correct ID if the Cluster AutoScaler attempts to remove the node when scaling the cluster.

The Cluster Autoscaler will begin to successfully scale down nodes once machine IDs match VM IDs.
To prevent a node being scaled down, the following annotation needs to be added to the node:

.. code-block:: bash

  kubectl annotate node <node-name> cluster-autoscaler.kubernetes.io/scale-down-disabled=true

This will indicate to CA that this node cannot be removed from the cluster when scaling down.


Cluster Autoscaler Deployment
------------------------------

The deployment of the CA on the cluster will be similar to the following:

.. code-block:: bash

  $ kubectl describe deployment cluster-autoscaler -n kube-system
  Name:                   cluster-autoscaler
  Namespace:              kube-system
  CreationTimestamp:      Mon, 16 Nov 2020 12:55:53 +0000
  Labels:                 app=cluster-autoscaler
  Annotations:            deployment.kubernetes.io/revision: 1
                          kubectl.kubernetes.io/last-applied-configuration:
                            {"apiVersion":"apps/v1","kind":"Deployment","metadata":{"annotations":{},"labels":{"app":"cluster-autoscaler"},"name":"cluster-autoscaler"...
  Selector:               app=cluster-autoscaler
  Replicas:               1 desired | 1 updated | 1 total | 1 available | 0 unavailable
  StrategyType:           RollingUpdate
  MinReadySeconds:        0
  RollingUpdateStrategy:  25% max unavailable, 25% max surge
  Pod Template:
    Labels:           app=cluster-autoscaler
    Service Account:  cluster-autoscaler-account
    Containers:
     cluster-autoscaler:
      Image:      docker.io/openstackmagnum/cluster-autoscaler:v1.0
      Port:       <none>
      Host Port:  <none>
      Command:
        ./cluster-autoscaler
        --alsologtostderr
        --cloud-provider=magnum
        --cluster-name=686f9fa1-eb56-4c23-9afd-67a79c283736
        --cloud-config=/config/cloud-config
        --nodes=1:4:default-worker
        --scale-down-unneeded-time=10m
        --scale-down-delay-after-failure=3m
        --scale-down-delay-after-add=10m
      Environment:  <none>
      Mounts:
        /config from cloud-config (ro)
        /etc/kubernetes from ca-bundle (ro)
    Volumes:
     ca-bundle:
      Type:        Secret (a volume populated by a Secret)
      SecretName:  ca-bundle
      Optional:    false
     cloud-config:
      Type:        Secret (a volume populated by a Secret)
      SecretName:  cluster-autoscaler-cloud-config
      Optional:    false
  Conditions:
    Type           Status  Reason
    ----           ------  ------
    Progressing    True    NewReplicaSetAvailable
    Available      True    MinimumReplicasAvailable
  OldReplicaSets:  cluster-autoscaler-8669c48d54 (1/1 replicas created)
  NewReplicaSet:   <none>
  Events:          <none>


We can see in the ``Command`` can change the time the autoscaler waits before determining that a node is unneeded and should be scaled down.
We can also change the delay time between adding nodes during scaling up and the amount of time to wait after scaling down fails.



Example: A Cluster Scaling Up
------------------------------

Let's have a cluster that has CA enabled and consists of one master node and one node.
If the worker node is cordoned and nginx pods still need to be scheduled, the CA will send an OpenStack request to resize the cluster and
increase the node count from 1 to 2 in order to have a node available to schedule a node. This can be seen in the container or pod logs for the CA:

.. code-block:: bash

  2020-11-16T11:00:13.721916753Z  I1116 11:00:13.721164       1 scale_up.go:689] Scale-up: setting group default-worker size to 2
  2020-11-16T11:00:21.441786855Z  I1116 11:00:21.441504       1 magnum_nodegroup.go:101] Increasing size by 1, 1->2
  2020-11-16T11:00:59.763966729Z  I1116 11:00:59.763422       1 magnum_nodegroup.go:67] Waited for cluster UPDATE_IN_PROGRESS status

You should see the stack for the cluster being updated on OpenStack and see the node visible in the cluster:

.. code-block:: bash

  ssh -i <mykey.pem> fedora@<master-node-ip>

  kubectl get nodes

  NAME                    STATUS                     ROLES    AGE     VERSION
  cluster-test-master-0   Ready                      master   2d20h   v1.14.3
  cluster-test-node-1     Ready,SchedulingDisabled   <none>   2d20h   v1.14.3
  cluster-test-node-2     NotReady                   <none>   0s      v1.14.3

  #Here we can see that the new node has been spun up and it being set up

  #After the node has been configured it reports that it is ready

  kubectl get nodes

  NAME                    STATUS                     ROLES    AGE     VERSION
  cluster-test-master-0   Ready                      master   2d20h   v1.14.3
  cluster-test-node-1     Ready,SchedulingDisabled   <none>   2d20h   v1.14.3
  cluster-test-node-2     Ready                      <none>   0s      v1.14.3


References
------------

https://docs.openstack.org/magnum/train/user/

https://github.com/kubernetes/autoscaler/tree/master/cluster-autoscaler/cloudprovider/magnum
