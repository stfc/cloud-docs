.. _swift_python_sdk:

===================================
Object Storage using the Python SDK
===================================

.. note::

    See :doc:`../Reference/PythonSDK` for how to set up using the Python SDK.


.. _swift_sdk_containers:

----------
Containers
----------

Containers for an account can be interacted with using a ``Connection`` instance, ``conn``, and the Object Store service.
Some examples are given below.

.. note::

    Containers can typically be passed into functions using either a ``Container`` instance or the container name as a string.


Listing containers for an account:

.. code-block:: python

    for cont in conn.object_store.containers():
        print(cont)


Creating a new container:

.. code-block:: python

    cont = conn.object_store.create_container("CONTAINER_1")


Deleting a container using its name:

.. code-block:: python

    conn.object_store.delete_container("CONTAINER_1")


.. note::

    The container must be empty before deletion.
    This command will not raise an error if the container specified is not found unless ``ignore_missing`` is set to ``False``.


Getting metadata for a container by passing a ``Container`` instance:

.. code-block:: python

    cont = conn.object_store.get_container_metadata(cont)
    print(cont)


Setting system metadata for a container, such as the read and write access, as well as an example custom property, by passing a ``Container`` instance:

.. code-block:: python

    conn.object_store.set_container_metadata(cont, write_ACL="example_project:example_user", read_ACL=".r:*,.rlistings", example_key="example_value")


.. note::

    System metadata is set using system keys: ``content_type``, ``is_content_type_detected``, ``versions_location``, ``read_ACL``, ``write_ACL``, ``sync_to``, ``sync_key``.


Deleting metadata for a container by passing a ``Container`` instance:

.. code-block:: python

    conn.object_store.delete_container_metadata(cont, ["example_key", "write_ACL", "read_ACL"])


.. note::

    Alternatively, ``set_container_metadata`` can be used with values set to ``""`` to delete custom and system metadata.


.. note::

    When setting and deleting custom metadata, capitalisation is ignored, and when deleting custom metadata, '-' and '_' are treated equivalently.
    This is not the case for system metadata.


.. _swift_sdk_objects:

-------
Objects
-------

Objects in a container can be interacted with using a ``Connection`` instance, ``conn``, and the Object Store service.
Some examples are given below.

.. note::

    Similarly to interacting with containers, objects can typically be specified using either an ``Object`` instance or the object and container names as strings.


Listing objects in a container by passing the container name:

.. code-block:: python

    objs = conn.object_store.objects("CONTAINER_1")
    for obj in objs:
        print(obj)


In the example above, ``objs`` is a generator object. Specific ``Object`` instances can be obtained from this in a number of ways, such as list comprehension:

.. code-block:: python

    obj_1 = [obj for obj in objs if obj.name=="FILE_1.txt"][0]


Objects can also be accessed directly using the container name and file name to return an ``Object`` instance:

.. code-block:: python

    obj_2 = conn.object_store.get_object("FILE_1.txt", "CONTAINER_1")


Equivalently:

.. code-block:: python

    obj_2 = conn.object_store.get_object_metadata("FILE_1.txt", "CONTAINER_1")


.. note::

    The ``Object`` instance returned by the two examples above (``obj_2``) differs slightly to that obtained using ``conn.object_store.objects()`` (``obj_1``).

    For example, the file name can be obtained via the ``name`` or ``id`` attributes of ``obj_1``, but only the ``id`` attribute of ``obj_2``.
    However, ``obj_2`` includes metadata not included in ``obj_1``, such as ``accept-ranges`` and ``x-timestamp``.


Specific objects can also be accessed via a ``Connection`` instance by passing the container name and file name. This will return a tuple, containing (headers, body) for the object specified:

.. code-block:: python

    obj_tuple = conn.get_object('CONTAINER_1', 'FILE_1.txt')


Similarly, using a ``Connection`` instance, container name and file name, a ``Response`` object can be returned, which stores the object ``headers`` and ``content`` as attributes:

.. code-block:: python

    response = conn.get_object_raw('CONTAINER_1', 'FILE_1.txt')


Getting metadata for a container using an ``Object`` instance (in the form of either ``obj_1`` or ``obj_2``):

.. code-block:: python

    obj = conn.object_store.get_object_metadata(obj)
    print(obj)


.. note::

    If an object in the form of ``obj_1`` is passed to ``get_object_metadata``, the object returned will include all the attributes of both ``obj_1`` and ``obj_2``.


Downloading an object's contents using an ``Object`` instance (in the form of either ``obj_1`` or ``obj_2``):

.. code-block:: python

    file_1 = conn.object_store.download_object(obj)


Alternatively, downloading contents using a ``Response`` object:

.. code-block:: python

    file_1 = response.content


In the two examples above, ``file_1`` will store the file contents as a ``bytes`` object. This can be written out in a number of ways, such as:

.. code-block:: python

    with open("SAVED_FILE_1.txt", "wb") as binary_file:
        binary_file.write(file_1)


Saving contents directly, without storing an intermediate ``Object`` or ``Response`` object:

.. code-block:: python

    conn.get_object('CONTAINER_1', 'FILE_1.txt', outfile="SAVED_FILE_1.txt")


Uploading a new object:

.. code-block:: python

    new_obj = conn.object_store.upload_object(container="CONTAINER_1",
                                            name="FILE_1.txt",
                                            data="Hello, world!")


Deleting an object using the container and file names:

.. code-block:: python

    conn.object_store.delete_object("FILE_1.txt", container="CONTAINER_1")


.. note::

    An error will not be raised if the object specified is not found unless ``ignore_missing`` is set to ``False``.


Setting system and custom metadata for an object by passing an ``Object`` instance:

.. code-block:: python

    conn.object_store.set_object_metadata(obj, delete_after="3000", example_key="example_value")


.. note::

    System metadata is set using system keys: ``content_type``, ``content_encoding``, ``content_disposition``, ``delete_after``, ``delete_at``, ``is_content_type_detected``.

    ``delete_at`` is also set automatically by setting ``delete_after``.


Deleting custom metadata for an object using the file and container names:

.. code-block:: python

    conn.object_store.delete_object_metadata("FILE_1.txt", "CONTAINER_4", ["example-key"])


.. note::

    Alternatively, ``set_object_metadata`` can be used with values set to ``""`` to delete custom metadata.
    However, neither option will delete system metadata.


.. note::

    When deleting custom metadata, the key should be in lower case, and underscores, '_', in the original key name should be replaced with dashes, '-'.



.. _swift_sdk_alternative:

-----------
swiftclient
-----------

An alternative to openstacksdk is swiftclient, which comprises a command line tool (see :ref:`swift_cli`) and two separate APIs, `SwiftService` and `Connection`, for accessing swift programmatically.

This can be installed using:

.. code-block:: bash

    pip install python-swiftclient


.. _swift_sdk_swiftservice:

SwiftService
------------

Below are two examples to illustrate the use of the swiftclient.SwiftService API.

.. note::

    There are several authentication mechanisms available, including setting environment variables that are automatically passed to your `SwiftService` instance. See :ref:`Use-OpenStack-CLI` for how to set these up.


Listing containers for your account:

.. code-block:: python

    import logging
    from swiftclient.service import SwiftService, SwiftError

    logging.basicConfig(level=logging.ERROR)
    logging.getLogger("requests").setLevel(logging.CRITICAL)
    logging.getLogger("swiftclient").setLevel(logging.CRITICAL)
    logger = logging.getLogger(__name__)

    with SwiftService() as swift:
        try:
            list_parts_gen = swift.list()
            for page in list_parts_gen:
                if page["success"]:
                    for item in page["listing"]:
                        i_name = item["name"]
                        i_size = int(item["bytes"])
                        i_count = int(item["count"])
                        print(f"{i_name} [size: {i_size}] [count: {i_count}]")
        except SwiftError as e:
            logger.error(e.value)


Listing and downloading all text files in a container:

.. code-block:: python

    import logging
    from swiftclient.service import SwiftService, SwiftError

    logging.basicConfig(level=logging.ERROR)
    logging.getLogger("requests").setLevel(logging.CRITICAL)
    logging.getLogger("swiftclient").setLevel(logging.CRITICAL)
    logger = logging.getLogger(__name__)

    def is_txt(obj):
        return (
            obj["name"].lower().endswith('.txt') or
            obj["content_type"] == 'text/plain'
        )

    container = "CONTAINER_1"

    with SwiftService() as swift:
        try:
            list_options = {"prefix": "archive_2016-01-01/"}
            list_parts_gen = swift.list(container=container)
            for page in list_parts_gen:
                if page["success"]:
                    objects = [
                        obj["name"] for obj in page["listing"] if is_txt(obj)
                    ]
                    for down_res in swift.download(
                            container=container,
                            objects=objects):
                        if down_res['success']:
                            print(f"'{down_res['object']}' downloaded")
                        else:
                            print(f"'{down_res['object']}' download failed")
                else:
                    raise page["error"]
        except SwiftError as e:
            logger.error(e.value)


.. warning::

    The ``content_type`` object key may not exist for objects, such as placeholders, which can lead to errors if the above code is run.


.. _swift_sdk_connection:

Connection
------------

Below are two examples to illustrate the use of the swiftclient.Connection API.

.. note::

    There are a number of kwarg combinations that can be used when creating a `Connection` instance for authentication.


Listing the response headers and containers for your account:

.. code-block:: python

    from swiftclient.client import Connection
    from keystoneauth1 import session
    from keystoneauth1.identity import v3

    # Create a password auth plugin
    auth = v3.Password(auth_url='https://openstack.stfc.ac.uk:5000/v3',
                        username='example_user',
                        password='example_passwprd',
                        project_id='example_id',
                        user_domain_name="example_domain_name")

    # Create session
    keystone_session = session.Session(auth=auth)

    # Create swiftclient Connection
    conn = Connection(session=keystone_session)

    resp_headers, containers = conn.get_account()
    print(f"Response headers: {resp_headers}")
    for container in containers:
        c_name = container["name"]
        c_count = int(container["count"])
        c_size = int(container["bytes"])
        print(f"{c_name} [size: {c_size}] [count: {c_count}]")


Deleting an object:

.. code-block:: python

    from swiftclient.client import Connection, ClientException
    from keystoneauth1 import session
    from keystoneauth1.identity import v3

    # Create a password auth plugin
    auth = v3.Password(auth_url='https://openstack.stfc.ac.uk:5000/v3',
                        username='example_user',
                        password='example_passwprd',
                        project_id='example_id',
                        user_domain_name="example_domain_name")

    # Create session
    keystone_session = session.Session(auth=auth)

    # Create swiftclient Connection
    conn = Connection(session=keystone_session)

    obj = 'FILE_1.txt'
    container = 'CONTAINER_1'
    try:
        conn.delete_object(container, obj)
        print("Successfully deleted the object")
    except ClientException as e:
        print(f"Failed to delete the object with error: {e}")


----------
References
----------

https://docs.openstack.org/openstacksdk/train/

https://docs.openstack.org/openstacksdk/train/user/resources/object_store/v1/container.html

https://docs.openstack.org/openstacksdk/train/user/resources/object_store/v1/obj.html

https://docs.openstack.org/openstacksdk/train/user/proxies/object_store.html

https://docs.openstack.org/openstacksdk/train/user/connection.html

https://docs.openstack.org/openstacksdk/train/user/guides/object_store.html

https://docs.openstack.org/python-swiftclient/train/introduction.html
