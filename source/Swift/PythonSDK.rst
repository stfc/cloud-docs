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


Get metadata for a container by passing its name:

.. code-block:: python

    cont = conn.object_store.get_container_metadata("CONTAINER_1")
    print(cont)


Alternatively, get metadata for a container by passing a ``Container`` object:

.. code-block:: python

    cont = conn.object_store.get_container_metadata(cont)
    print(cont)


Delete a container using its name:

.. code-block:: python

    conn.object_store.delete_container("CONTAINER_2")


Delete a container using a ``Container`` object:

.. code-block:: python

    conn.object_store.delete_container(cont)


In the two examples above, the container must be empty before deletion.
Also note that this will not raise an error if the container specified is not found unless ``ignore_missing`` is set to ``False``.


.. _swift_sdk_objects:

Objects
-------

List objects in a container by passing a container name:

.. code-block:: python

    objs = conn.object_store.objects("CONTAINER_1")
    for obj in objs:
        print(obj)


Alternatively, list objects in a container by passing a ``Container`` object:

.. code-block:: python

    objs = conn.object_store.objects(cont)
    for obj in objs:
        print(obj)


In the two examples above, ``objs`` is a generator object. Specific ``Object`` objects can be obtained from this in a number of ways, such as list comprehension:

.. code-block:: python

    objs = conn.object_store.objects("CONTAINER_1")
    obj_1 = [obj for obj in objs if obj.name=="FILE_1.txt"][0]


Specific objects can also be accessed via the ``Connection`` object, passing the container name and file name. This will return a tuple, containing (headers, body) for the object specified:
:

.. code-block:: python

    obj_tuple = conn.get_object('CONTAINER_1', 'FILE_1.txt')


Similarly, using the ``Connection`` object, container name and file name, a ``Response`` for the object can be returned, which stores the object ``headers`` and ``content`` as attributes:

.. code-block:: python

    response = conn.get_object_raw('CONTAINER_1', 'FILE_1.txt')


Objects can also be accessed via an ``Object Store`` object, using the container name and file name to return an ``Object`` object:

.. code-block:: python

    obj_2 = conn.object_store.get_object("FILE_1.txt", "CONTAINER_1")


Alternatively:

.. code-block:: python

    obj_2 = conn.object_store.get_object_metadata("FILE_1.txt", "CONTAINER_1")


The two examples above are equivalent to each other.

However, the returned ``Object`` object returned (``obj_2``) differs slightly to those obtained via ``conn.object_store.objects()`` (``obj_1``).
For example, the file name can be obtained via the ``name`` or ``id`` attributes of ``obj_1``, but only the ``id`` attribute of ``obj_2``.


An object's contents can be downloaded using ``Object`` objects (in the form of both ``obj_1`` and ``obj_2``):

.. code-block:: python

    file_1 = conn.object_store.download_object(obj)


Alternatively, using a ``Response`` object:

.. code-block:: python

    file_1 = response.content


In the two examples above, ``file_1`` will store the file contents as a ``bytes`` object. This can be written out in a number of ways, such as:

.. code-block:: python

    with open("SAVED_FILE_1.txt", "wb") as binary_file:
        binary_file.write(file_1)


Objects can also be saved directly, without storing an intermediate ``Object`` or ``Response`` object:

.. code-block:: python

    conn.get_object('CONTAINER_1', 'FILE_1.txt', outfile="SAVED_FILE_1.txt")


Upload a new file:

.. code-block:: python

    new_obj = conn.object_store.upload_object(container="CONTAINER_1",
                                            name="FILE_1.txt",
                                            data="Hello, world!")


References
----------

https://docs.openstack.org/openstacksdk/train/user/resources/object_store/v1/container.html

https://docs.openstack.org/openstacksdk/train/user/resources/object_store/v1/obj.html

https://docs.openstack.org/openstacksdk/train/user/proxies/object_store.html

https://docs.openstack.org/openstacksdk/train/user/connection.html

https://docs.openstack.org/openstacksdk/train/user/guides/object_store.html
