.. _introducing_swift:

=====
Swift
=====


.. _swift_introduction:

Introduction
------------

Swift is a highly available, distributed, eventually consistent object/blob store. Swift allows lots of data to be stored efficiently, safely, and cheaply. It's built for scale and optimized for durability, availability, and concurrency across the entire data set. Swift is ideal for storing unstructured data that can grow without bound.

Files and data are uploaded as **objects**, which are organised into **containers**.

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

Access Control List (ACL):
Used to control access to objects within a container.
An ACL cannot be stored with individual objects.


References
----------

https://www.openstack.org/software/releases/train/components/swift

https://docs.openstack.org/swift/train/

https://access.redhat.com/documentation/en-us/red_hat_gluster_storage/3.1/html/administration_guide/chap-managing_object_store

https://access.redhat.com/documentation/en-us/red_hat_gluster_storage/3.1/html/administration_guide/components_of_object_storage

https://docs.openstack.org/swift/train/api/object_api_v1_overview.html

https://docs.openstack.org/swift/train/api/pseudo-hierarchical-folders-directories.html

https://docs.openstack.org/swift/train/overview_acl.html
