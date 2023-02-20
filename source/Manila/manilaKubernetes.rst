Using Shares on Kubernetes Clusters
######################################

.. warning::

    OpenStack Manila is currently in technical preview on STFC Cloud. If you have any feedback or suggestions, please sent it to cloud-support@stfc.ac.uk

    This feature for using Shares with Kubernetes is also currently being tested by the Cloud Team.


To use shares on Kubernetes clusters we need the following ``csi`` installed and deployed onto the cluster:
    - ``manila-csi``: https://github.com/kubernetes/cloud-provider-openstack/blob/master/docs/manila-csi-plugin/using-manila-csi-plugin.md

    - ``ceph-csi``: https://github.com/ceph/ceph-csi

The ``manila-csi`` is designed to be lightweight and only handles the interaction between Kubernetes and OpenStack to create, delete, and access shares. 
The ``ceph-csi`` handles the CephFS side of the filesystem on the Kubernetes cluster. See https://github.com/kubernetes/cloud-provider-openstack/blob/master/docs/manila-csi-plugin/developers-csi-manila.md#notes-on-design  for more information on how manila-csi is designed.

Deploying onto the Cluster 
-----------------------------
Both the ``manila-csi`` and ``ceph-csi`` can be deployed to a Kubernetes cluster using a Helm chart. 

ceph-csi
~~~~~~~~~~~

The ``ceph-csi`` can be deployed in the following way:

.. code-block:: bash 

    # Add the chart repository 
    helm repo add ceph-csi https://ceph.github.io/csi-charts
    # Create namespace for the chart
    kubectl create namespace ceph-csi-cephfs
    # Install the chart
    helm install --namespace "ceph-csi-cephfs" "ceph-csi-cephfs" ceph-csi/ceph-csi-cephfs


You should then be able to see the following pods in the `ceph-csi-cephfs` namespace:

.. code-block:: bash 

    $ kubectl get pods -n ceph-csi-cephfs
    NAME                                           READY   STATUS    RESTARTS   AGE
    ceph-csi-cephfs-nodeplugin-6554g               3/3     Running   0          17d
    ceph-csi-cephfs-nodeplugin-6xcfc               3/3     Running   0          17d
    ceph-csi-cephfs-nodeplugin-ddt84               3/3     Running   0          17d
    ceph-csi-cephfs-nodeplugin-h97j5               3/3     Running   0          17d
    ceph-csi-cephfs-nodeplugin-hxj95               3/3     Running   0          17d
    ceph-csi-cephfs-nodeplugin-j65fp               3/3     Running   0          17d
    ceph-csi-cephfs-nodeplugin-ltt7q               3/3     Running   0          17d
    ceph-csi-cephfs-nodeplugin-mhwr8               3/3     Running   0          17d
    ceph-csi-cephfs-nodeplugin-wrjsk               3/3     Running   0          17d
    ceph-csi-cephfs-nodeplugin-zfl8w               3/3     Running   0          17d
    ceph-csi-cephfs-provisioner-765854df4c-csdft   5/5     Running   0          17d
    ceph-csi-cephfs-provisioner-765854df4c-nzp77   5/5     Running   0          17d
    ceph-csi-cephfs-provisioner-765854df4c-qznz4   5/5     Running   0          17d



manila-csi
~~~~~~~~~~~~~~~~~~

Before deploying the Helm chart for ``manila-csi``, we need to modify the ``values.yaml`` file in order to deploy 
the csi supporting ``CephFS`` shares:

.. code-block:: yaml

    # manila-csi-values.yaml
    nameOverride: ""
    fullNameOverride: ""

    # Base name of the CSI Manila driver
    driverName: manila.csi.openstack.org

    # Enabled Manila share protocols
    shareProtocols:
    - protocolSelector: CEPHFS
    fsGroupPolicy: None
    fwdNodePluginEndpoint:
        dir: /var/lib/kubelet/plugins/cephfs.csi.ceph.com
        sockFile: csi.sock

Then we can deploy the chart: 

.. code-block:: bash 

    # add helm repo
    helm repo add cpo https://kubernetes.github.io/cloud-provider-openstack
    helm repo update

    # install helm chart with values.yaml file 'manila-csi-values.yaml'
    helm install manila-csi -f manila-csi-values.yaml cpo/openstack-manila-csi

Then you should see pods similar to the following being created onto the cluster (for this example the chart has been deployed into the default namespace):

.. code-block:: bash 

    $ kubectl get pod
    NAME                                                              READY   STATUS    RESTARTS   AGE
    manila-csi-openstack-manila-csi-controllerplugin-0                4/4     Running   0          14d
    manila-csi-openstack-manila-csi-nodeplugin-6plbt                  2/2     Running   0          14d
    manila-csi-openstack-manila-csi-nodeplugin-9dxrl                  2/2     Running   0          14d
    manila-csi-openstack-manila-csi-nodeplugin-cjhvp                  2/2     Running   0          14d
    manila-csi-openstack-manila-csi-nodeplugin-f72s9                  2/2     Running   0          14d
    manila-csi-openstack-manila-csi-nodeplugin-ghfmx                  2/2     Running   0          14d
    manila-csi-openstack-manila-csi-nodeplugin-h99sx                  2/2     Running   0          14d
    manila-csi-openstack-manila-csi-nodeplugin-k5g9k                  2/2     Running   0          14d
    manila-csi-openstack-manila-csi-nodeplugin-mlvpp                  2/2     Running   0          14d
    manila-csi-openstack-manila-csi-nodeplugin-qpd9m                  2/2     Running   0          14d
    manila-csi-openstack-manila-csi-nodeplugin-slktx                  2/2     Running   0          14d

Next step is to manually create the secret for the manila-csi to use in order to interact with OpenStack. The secrets template looks similar to this:

.. code-block:: yaml 

    # manila_csi_secret.yaml
    apiVersion: v1
    kind: Secret
    metadata:
    name: csi-manila-secret
    namespace: kube-system # or another namespace - pvc will require the namespace where the secret is stored
    StringData:
    # mandatory
    os-authURL: "<auth-url>"
    os-region: "<region>"
    # authentication using application credentials
    os-applicationCredentialID: "<application-credential-id>"
    os-applicationCredentialSecret: "<application-credential-secret>"
    # authentication using user credentials  
    os-projectID: "<project-id>"
    os-userName: "<username>"
    os-password: "<password>"
    os-domainName: "<domain>"

.. warning:: 

    Although creating the secret using a service account credential has been tested, it is not recommended to use when preparing the secret for the manila-csi on Kubernetes. 
    Ideally an application credential should be used. 

The secret can then be created on the cluster using:

.. code-block:: bash 

    kubectl apply -f manila_csi_secret.yaml


Examples: Provisioning Manila Shares onto the Cluster
---------------------------------------------------------

.. note::

    The following examples are following the examples from https://github.com/kubernetes/cloud-provider-openstack/tree/master/examples/manila-csi-plugin


Static Share Provisioning 
~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. note::

    If a default storage class has been set up in the cluster, then ``storageClass: ""`` needs to be included in the ``pvc.yaml`` file. 
    Otherwise, the cluster will default to attempting to make a pvc based on the cluster's default storage class.

- Create a persistent volume claim (PVC) and persistent volume:

.. code-block:: yaml 

    apiVersion: v1
    kind: PersistentVolume
    metadata:
      name: preprovisioned-cephfs-share
      labels:
        name: preprovisioned-cephfs-share
    spec:
      accessModes:
      - ReadWriteMany
      capacity:
        storage: 1Gi
      csi:
        driver: cephfs.manila.csi.openstack.org
        volumeHandle: preprovisioned-cephfs-share
        nodeStageSecretRef:
          name: csi-manila-secret
          namespace: kube-system
        nodePublishSecretRef:
          name: csi-manila-secret
          namespace: kube-system
        volumeAttributes:
          shareID: <share-id>
          shareAccessID: <access-rule-id>
    ---
    apiVersion: v1
    kind: PersistentVolumeClaim
    metadata:
      name: existing-cephfs-share-pvc
    spec:
      accessModes:
      - ReadWriteMany
      storageClassName: "" # to override any default storageclass in cluster
      resources:
        requests:
          storage: 1Gi
      selector:
        matchExpressions:
        - key: name
          operator: In
          values: ["preprovisioned-cephfs-share"]


You should be able to create the persistent volume claim created successfully here:

.. code-block:: bash 

  $ kubectl apply -f pvc_pv.yaml

  $ kubectl get pvc
  
  NAME                              STATUS    VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS        AGE
  existing-cephfs-share-pvc         Bound     preprovisioned-cephfs-share                1Gi        RWX                                19h

  kubectl describe pvc existing-cephfs-share-pvc


  $ kubectl describe pvc existing-cephfs-share-pvc
  Name:          existing-cephfs-share-pvc
  Namespace:     default
  StorageClass:
  Status:        Bound
  Volume:        preprovisioned-cephfs-share
  Labels:        <none>
  Annotations:   pv.kubernetes.io/bind-completed: yes
                 pv.kubernetes.io/bound-by-controller: yes
  Finalizers:    [kubernetes.io/pvc-protection]
  Capacity:      1Gi
  Access Modes:  RWX
  VolumeMode:    Filesystem
  Used By:       existing-cephfs-share-pod
  Events:        <none>

  $ kubectl get pv
  NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                                STORAGECLASS        REASON   AGE
  preprovisioned-cephfs-share                1Gi        RWX            Retain           Bound    default/existing-cephfs-share-pvc                                 19h

  $ kubectl describe pv preprovisioned-cephfs-share
  Name:            preprovisioned-cephfs-share
  Labels:          name=preprovisioned-cephfs-share
  Annotations:     pv.kubernetes.io/bound-by-controller: yes
  Finalizers:      [kubernetes.io/pv-protection]
  StorageClass:
  Status:          Bound
  Claim:           default/existing-cephfs-share-pvc
  Reclaim Policy:  Retain
  Access Modes:    RWX
  VolumeMode:      Filesystem
  Capacity:        1Gi
  Node Affinity:   <none>
  Message:
  Source:
      Type:              CSI (a Container Storage Interface (CSI) volume source)
      Driver:            cephfs.manila.csi.openstack.org
      FSType:
      VolumeHandle:      preprovisioned-cephfs-share
      ReadOnly:          false
      VolumeAttributes:      shareAccessID=SHARE_ACCESS_RULE_ID
                             shareID=SHARE_ID
  Events:                <none>


Dynamic Share Provisioning
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Shares can also be created on demand and attached to specific pods. 

First, a storage class needs to be created:

.. code-block:: yaml 

  apiVersion: storage.k8s.io/v1
  kind: StorageClass
  metadata:
    name: csi-manila-cephfs
  provisioner: cephfs.manila.csi.openstack.org
  allowVolumeExpansion: true
  parameters:
    # Manila share type
    type: cephfs

    csi.storage.k8s.io/provisioner-secret-name: csi-manila-secret
    csi.storage.k8s.io/provisioner-secret-namespace: kube-system
    csi.storage.k8s.io/controller-expand-secret-name: csi-manila-secret
    csi.storage.k8s.io/controller-expand-secret-namespace: kube-system
    csi.storage.k8s.io/node-stage-secret-name: csi-manila-secret
    csi.storage.k8s.io/node-stage-secret-namespace: kube-system
    csi.storage.k8s.io/node-publish-secret-name: csi-manila-secret
    csi.storage.k8s.io/node-publish-secret-namespace: kube-system

Then the storage class can be created from this template using:

.. code-block:: bash 

  kubectl apply -f storageclass.yaml 

Next, a persistent volume claim needs to be created in order to define the size of the share and the access mode.

.. code-block:: yaml 

  apiVersion: v1
  kind: PersistentVolumeClaim
  metadata:
    name: new-cephfs-share-pvc
  spec:
    accessModes:
      - ReadWriteMany
    resources:
      requests:
        storage: 1Gi
    storageClassName: csi-manila-cephfs

To create the pvc: 

.. code-block:: bash 

  kubectl apply -f dynamic_pvc.yaml 

Then, for example, if we want to have an nginx pod spun up with a new share attached we can use the following template:

.. code-block:: yaml 

  apiVersion: v1
  kind: Pod
  metadata:
    name: new-cephfs-share-pod
  spec:
    containers:
      - name: web-server
        image: nginx
        imagePullPolicy: IfNotPresent
        volumeMounts:
          - name: mypvc
            mountPath: /var/lib/www
    volumes:
      - name: mypvc
        persistentVolumeClaim:
          claimName: new-cephfs-share-pvc
          readOnly: false

Then we can create the pod and inspect it:

.. code-block:: bash 

  $ kubectl apply -f pod.yaml

  $ kubectl describe pod new-cephfs-share-pod
  Name:             new-cephfs-share-pod
  Namespace:        default
  Priority:         0
  Service Account:  default
  Node:             cloud-dev-md-nano-fxhmg/10.6.0.204
  Start Time:       Thu, 02 Feb 2023 14:21:56 +0000
  Labels:           <none>
  Annotations:      cni.projectcalico.org/containerID: d0741d0c1f5b8c4733c3a7c090dbaef9e6875508ab074c36cc671bfa51479494
                    cni.projectcalico.org/podIP: 192.168.232.10/32
                    cni.projectcalico.org/podIPs: 192.168.232.10/32
  Status:           Running
  IP:               192.168.232.10
  IPs:
    IP:  192.168.232.10
  Containers:
    web-server:
      Container ID:   containerd://12a76ece022f9cbc45b6d9af3e2c0efa38dafb4734a52dcdbceaa4f63caf174e
      Image:          nginx
      Image ID:       docker.io/library/nginx@sha256:b8f2383a95879e1ae064940d9a200f67a6c79e710ed82ac42263397367e7cc4e
      Port:           <none>
      Host Port:      <none>
      State:          Running
        Started:      Thu, 02 Feb 2023 14:22:10 +0000
      Ready:          True
      Restart Count:  0
      Environment:    <none>
      Mounts:
        /var/lib/www from mypvc (rw)
        /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-6l758 (ro)
  Conditions:
    Type              Status
    Initialized       True
    Ready             True
    ContainersReady   True
    PodScheduled      True
  Volumes:
    mypvc:
      Type:       PersistentVolumeClaim (a reference to a PersistentVolumeClaim in the same namespace)
      ClaimName:  new-cephfs-share-pvc
      ReadOnly:   false
    kube-api-access-6l758:
      Type:                    Projected (a volume that contains injected data from multiple sources)
      TokenExpirationSeconds:  3607
      ConfigMapName:           kube-root-ca.crt
      ConfigMapOptional:       <nil>
      DownwardAPI:             true
  QoS Class:                   BestEffort
  Node-Selectors:              <none>
  Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                               node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
  Events:                      <none>

In the OpenStack project where the application credential was created for, we should now see a new share has been created with a cephx access rule.