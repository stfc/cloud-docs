.. _swift_python_sdk:

===================================
Object Storage using the Python SDK
===================================

.. note::

    See :doc:`../Reference/PythonSDK` for how to set up using the Python SDK.


.. _swift_sdk_containers:

Containers
----------

List containers for the account:

.. code-block:: python

    for cont in conn.object_store.containers():
        print(cont)


Create a new container:

.. code-block:: python

    cont = conn.object_store.create_container(name="CONTAINER_1")


Get metadata for a container:

.. code-block:: python

    cont = conn.object_store.get_container_metadata("CONTAINER_1")
    print(cont)


References
----------

https://docs.openstack.org/openstacksdk/train/user/guides/object_store.html