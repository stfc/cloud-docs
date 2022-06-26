.. _introducing_swift:

=================
Introducing Swift
=================

Swift is a highly available, distributed, eventually consistent object/blob store.
Due to the flat structure of object storage, it is highly scalable, allowing large amounts of unstructured data to be stored efficiently.

Files and data are uploaded as **objects**, which are organised into **containers**.
Once created, object data cannot be modified without uploading the entire object again, making this storage unsuitable for frequently updated data.

.. note::

  Swift containers should not be confused with Docker containers. Containers may also be referred to as buckets or pools.


Although you cannot nest directories in object storage, a hierarchical structure can be simulated within a container through the inclusion of delimiters ('/') in object names.


.. _swift_terms:

Explanation of Terms
--------------------

Object Store:
This provides a system for data storage that enables users to access the same data, both as an object and as a file, simplifying management and controlling storage costs.

Account:
The top level of the Object Storage system hierarchy, defining a namespace for containers.
Users own all resources in their account.
Synonymous with a `project` or `tenant`.

Container:
This defines a namespace for objects, providing a storage compartment for and a way to organise data, similar to directories in a Linux system.
However, unlike directories, containers cannot be nested.
Data must be stored in a container, and so objects are created within containers.
Containers may also be referred to as buckets or pools.

Object:
The basic storage entity and any optional metadata that represents the data being stored.
When data is uploaded, the data is stored as-is, with no compression or encryption.

Folder/Pseudo-folder:
Similar to folders in a desktop operating system.
They are virtual collections defined by a common prefix in objects' names.
Placeholder objects may also exist to represent folders.

Eventually Consistent:
When data is updated, it is guaranteed that this will be reflected across all nodes eventually.
This allows high availability, but there is no guarantee for when this will happen, so changes to data may not be reflected in read requests immediately.

Unstructured Data:
Data that does not conform to, or cannot be easily organised into, a traditional relational database, including emails, videos and web pages.

Access Control List (ACL):
Used to control access to objects within a container.
An ACL cannot be stored with individual objects.


References
----------

https://www.openstack.org/software/releases/train/components/swift

https://docs.openstack.org/swift/train/

https://www.redhat.com/en/topics/data-storage/file-block-object-storage

https://access.redhat.com/documentation/en-us/red_hat_gluster_storage/3.1/html/administration_guide/chap-managing_object_store

https://access.redhat.com/documentation/en-us/red_hat_gluster_storage/3.1/html/administration_guide/components_of_object_storage

https://docs.openstack.org/swift/train/api/object_api_v1_overview.html

https://docs.openstack.org/swift/train/api/pseudo-hierarchical-folders-directories.html

https://docs.openstack.org/swift/train/overview_acl.html

https://www.ibm.com/cloud/learn/object-storage