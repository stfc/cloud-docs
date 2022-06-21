.. _swift_cli:

=========
Swift CLI
=========

In order to use all the commands provided from Swift, it is recommended that you have the following python package installed:

.. code-block:: bash

  pip install python-swiftclient


.. note::

    See :ref:`Use-OpenStack-CLI` for how to set up the Openstack command line client. This is the preferred client, but in some cases the Swift client provides coverage that is not yet supported.


To test whether the Swift client have been installed successfully and we can access Swift commands, we can try to list containers in our current project:

.. code-block:: bash

  swift list


This should return nothing if there are no containers in your project, or a list of containers similar to below:

.. code-block:: bash

  CONTAINER_1
  CONTAINER_2
  CONTAINER_3


We can also try to list containers using the Openstack client, to test that this has been installed successfully and that we can access Swift commands in the preferred manner:

.. code-block:: bash

  openstack container list


This should return an empty line if there are no containers in your project, or a table containing the names of containers similar to the table below:

.. code-block:: bash

  +-------------+
  | Name        |
  +-------------+
  | CONTAINER_1 |
  | CONTAINER_2 |
  | CONTAINER_3 |
  +-------------+


.. _swift_cli_commands:

Commands
--------

We are now able to run commands for working with containers and objects. Below is a list of some of the commands that can be used:

.. code-block:: bash

  # Openstack client commands are of the form:
  openstack container <commands>
  openstack object <commands>

  # Containers
  container create <container> # Create a new container
  container delete <container> # Delete a container
  container list # List containers
  container save <container> # Save a container's contents locally
  container set --property <key>=<value> <container> # Set a container's properties
  container show <container> # Display a container's details
  container unset --property <key> <container> # Unset a container's properties

  # Objects
  object create <container> <object> # Upload an object to a container
  object delete <container> <object> # Delete an object from a container
  object list <container> # List objects in a container
  object save <container> <object> # Save an object locally
  object set --property <key>=<value> <container> <object> # Set an object's properties
  object show <container> <object> # Display an object's details
  object unset --property <key> <container> <object> # Unset an object's properties

  # Object store
  object store account set --property <key>=<value> # Set an account's properties
  object store account show # Display an account's details
  object store account unset --property <key> # Unset an account's properties


.. code-block:: bash

  # Swift client commands are of the form:
  swift <commands>

  # Containers
  delete <container> # Delete a container, including all objects within
  download <container> # Save a container's contents locally
  list # List containers
  post <container> # Update a container's details, or create a container if it does not exist
  stat <container> # Display a container's details

  # Objects
  copy <container> <object> # Updates an object's metadata, or copies an object to a new destination
  delete <container> <object> # Delete an object from a container
  download <container> <object> # Save an object locally
  list <container> # List objects in a container
  post <container> <object> # Update an object's details
  stat <container> <file> # Display's an object's details
  upload <container> <object> # Upload an object to a container

  # Other
  auth # Display authentication variables
  capabilities # Displays cluster capabilities for an object
  delete --all # Deletes everything in the account
  download --all # Downloads everything in the account
  post # Update an account's details
  stat # Display information for the account
  tempurl <method> <time> <path> <key> # Generates a temporary URL for an object


.. warning::

  Equivalent commands using the Openstack and Swift clients may differ in behaviour. For example, ``openstack container delete <container>`` will fail unless the container is empty or ``-r`` is used, whereas ``swift delete <container>`` will delete the contents of the container, as well as the container itself.


Further details and options can be seen using ``openstack container <command> --help``, ``openstack object <command> --help`` and ``swift <command> --help``.


.. _swift_cli_list:

Listing Files
-------------

To list objects in ``CONTAINER_1``:

.. code-block:: bash

  openstack object list CONTAINER_1

  # This should return a table similar to:
  +---------------------+
  | Name                |
  +---------------------+
  | FILE_1.txt          |
  | FILE_2.txt          |
  | FOLDER_1/FILE_1.txt |
  +---------------------+


.. _swift_cli_information:

Getting Information About Your Account, Containers and Files
------------------------------------------------------------

To print information about your account:

.. code-block:: bash

  swift stat

  # This should return text similar to:
                                      Account: v1
                                   Containers: 3
                                      Objects: 9
                                        Bytes: 21
  Objects in policy "default-placement-bytes": 0
    Bytes in policy "default-placement-bytes": 0
     Containers in policy "default-placement": 3
        Objects in policy "default-placement": 9
          Bytes in policy "default-placement": 21
                            Meta Temp-Url-Key: MYKEY
                                     Meta Key: value
                                  X-Timestamp: 1654612569.67835
                  X-Account-Bytes-Used-Actual: 20480
                                   X-Trans-Id: tx000000000000008f227e6-00629f6259-218cd2a5a-default
                       X-Openstack-Request-Id: tx000000000000008f227e6-00629f6259-218cd2a5a-default
                                Accept-Ranges: bytes
                                 Content-Type: text/plain; charset=utf-8


To print basic information about ``CONTAINER_1``:

.. code-block:: bash

  openstack container show CONTAINER_1

  # This should return a table similar to:
  +--------------+-------------+
  | Field        | Value       |
  +--------------+-------------+
  | account      | v1          |
  | bytes_used   | 18          |
  | container    | CONTAINER_1 |
  | object_count | 3           |
  +--------------+-------------+

To print more detailed information about ``CONTAINER_1``:

.. code-block:: bash

  swift stat CONTAINER_1

  # This should return text similar to:
                        Account: v1
                      Container: CONTAINER_1
                        Objects: 3
                          Bytes: 18
                       Read ACL:
                      Write ACL:
                        Sync To:
                       Sync Key:
                    X-Timestamp: 1652205950.32071
  X-Container-Bytes-Used-Actual: 12288
               X-Storage-Policy: default-placement
                X-Storage-Class: STANDARD
                  Last-Modified: Fri, 13 May 2022 20:04:16 GMT
                     X-Trans-Id: tx0000000000000058f1bbc-006283a793-21529d22a-default
         X-Openstack-Request-Id: tx0000000000000058f1bbc-006283a793-21529d22a-default
                  Accept-Ranges: bytes
                   Content-Type: text/plain; charset=utf-8


To print basic information about ``FILE_1.txt`` in ``CONTAINER_1``:

.. code-block:: bash

  openstack object show CONTAINER_1 FILE_1.txt

  # This should return a table similar to:
  +----------------+----------------------------------+
  | Field          | Value                            |
  +----------------+----------------------------------+
  | account        | v1                               |
  | container      | CONTAINER_1                      |
  | content-length | 4                                |
  | content-type   | text/plain                       |
  | etag           | 0cbc6611f5540bd0809a388dc95a615b |
  | last-modified  | Tue, 17 May 2022 13:50:29 GMT    |
  | object         | FILE_1.txt                       |
  | properties     | Orig-Filename='FILE_1.txt'       |
  +----------------+----------------------------------+


To print further information about ``FILE_1.txt`` in ``CONTAINER_1``:

.. code-block:: bash

  swift stat CONTAINER_1 FILE_1.txt

  # This should return text similar to:
                 Account: v1
              Container: CONTAINER_1
                  Object: FILE_1.txt
            Content Type: text/plain
          Content Length: 4
          Last Modified: Tue, 17 May 2022 13:50:29 GMT
                    ETag: 0cbc6611f5540bd0809a388dc95a615b
      Meta Orig-Filename: FILE_1.txt
          Accept-Ranges: bytes
            X-Timestamp: 1652795429.54108
              X-Trans-Id: tx000000000000004ef007b-006283a908-218cd2a5a-default
  X-Openstack-Request-Id: tx000000000000004ef007b-006283a908-218cd2a5a-default


.. _swift_cli_create_containers:

Creating Containers
-------------------

Containers can be created using:

.. code-block:: bash

  openstack container create [-h] [-f {csv,json,table,value,yaml}] [-c COLUMN]
                                  [--quote {all,minimal,none,nonnumeric}] [--noindent] [--max-width <integer>]
                                  [--fit-width] [--print-empty] [--sort-column SORT_COLUMN]
                                  [--sort-ascending | --sort-descending]
                                  <container-name> [<container-name> ...]

  Create new container

  positional arguments:
    <container-name>
                          New container name(s)

  optional arguments:
    -h, --help            show this help message and exit

  output formatters:
    output formatter options

    -f {csv,json,table,value,yaml}, --format {csv,json,table,value,yaml}
                          the output format, defaults to table
    -c COLUMN, --column COLUMN
                          specify the column(s) to include, can be repeated to show multiple columns
    --sort-column SORT_COLUMN
                          specify the column(s) to sort the data (columns specified first have a priority, non-existing columns are
                          ignored), can be repeated
    --sort-ascending      sort the column(s) in ascending order
    --sort-descending     sort the column(s) in descending order


For example:

.. code-block:: bash

  openstack container create CONTAINER_1

  # This should return a table similar to:
  +---------+-------------+------------------------------------------------------+
  | account | container   | x-trans-id                                           |
  +---------+-------------+------------------------------------------------------+
  | v1      | CONTAINER_1 | tx00000000000000384233c-006273aa93-21531bd94-default |
  +---------+-------------+------------------------------------------------------+


.. note::

  Containers created using the Openstack client will not be publicly accessible. This can be changed via the GUI, or by :ref:`updating the container's metadata <swift_cli_editing_containers>`.


.. _swift_cli_upload_objects:

Uploading Files
---------------

Objects can be uploaded into containers using:

.. code-block:: bash

  openstack object create [-h] [-f {csv,json,table,value,yaml}] [-c COLUMN]
                               [--quote {all,minimal,none,nonnumeric}] [--noindent] [--max-width <integer>]
                               [--fit-width] [--print-empty] [--sort-column SORT_COLUMN] [--sort-ascending | --sort-descending]
                               [--name <name>]
                               <container> <filename> [<filename> ...]

  Upload object to container

  positional arguments:
    <container>   Container for new object
    <filename>    Local filename(s) to upload

  optional arguments:
    -h, --help            show this help message and exit
    --name <name>
                          Upload a file and rename it. Can only be used when uploading a single object

  output formatters:
    output formatter options

    -f {csv,json,table,value,yaml}, --format {csv,json,table,value,yaml}
                          the output format, defaults to table
    -c COLUMN, --column COLUMN
                          specify the column(s) to include, can be repeated to show multiple columns
    --sort-column SORT_COLUMN
                          specify the column(s) to sort the data (columns specified first have a priority, non-existing columns are
                          ignored), can be repeated
    --sort-ascending      sort the column(s) in ascending order
    --sort-descending     sort the column(s) in descending order

  CSV Formatter:
    --quote {all,minimal,none,nonnumeric}
                          when to include quotes, defaults to nonnumeric

  json formatter:
    --noindent            whether to disable indenting the JSON

  table formatter:
    --max-width <integer>
                          Maximum display width, <1 to disable. You can also use the CLIFF_MAX_TERM_WIDTH environment variable, but the
                          parameter takes precedence.
    --fit-width           Fit the table to the display width. Implied if --max-width greater than 0. Set the environment variable
                          CLIFF_FIT_WIDTH=1 to always enable
    --print-empty         Print empty table if there is no data to show.


Multiple files may be uploaded simultaneously by listing then after the container name:

.. code-block:: bash

  openstack object create CONTAINER_1 FILE_1.txt FILE_2.txt

  # This should return a table similar to:
  +------------+-------------+----------------------------------+
  | object     | container   | etag                             |
  +------------+-------------+----------------------------------+
  | FILE_1.txt | CONTAINER_1 | ff22941336956098ae9a564289d1bf1b |
  | FILE_2.txt | CONTAINER_1 | 9c8c1df0ae41d9a418d596e7ddfefb3b |
  +------------+-------------+----------------------------------+


.. note::

  The name of the object uploaded will include its relative local path, unless otherwise specified using the ``--name`` option. For example, if ``./FOLDER_1/FILE_1.txt`` is uploaded, it will be named ``FOLDER_1/FILE_1.txt`` in the container by default.


.. _swift_cli_create_folders:

Creating Folders
----------------

Folders can be created when uploading a file. For example, ``FOLDER_1`` and ``FOLDER_2`` can be created with the following:

.. code-block:: bash

  openstack object create CONTAINER_1 FOLDER_1/FOLDER_2/FILE_1.txt

  # This should return a table similar to:
  +------------------------------+-------------+----------------------------------+
  | object                       | container   | etag                             |
  +------------------------------+-------------+----------------------------------+
  | FOLDER_1/FOLDER_2/FILE_1.txt | CONTAINER_1 | 2205e48de5f93c784733ffcca841d2b5 |
  +------------------------------+-------------+----------------------------------+


To create an empty folder in a container, a local empty folder can be uploaded using the Swift client:

.. code-block:: bash

  swift upload CONTAINER_1 FOLDER_1/

  # This should return the name of the created folder:
  FOLDER_1/


.. note::

  The trailing `/` must be included to create a folder in this way.


.. _swift_cli_save_containers:

Saving Containers
-----------------

The full contents of a container can be saved using the Openstack client. For example:

.. code-block:: bash

  openstack container save CONTAINER_1


All files will be downloaded to your current directory, with directories implied by object names being created as necessary to recreate the structure.

For example, saving the following container would save ``FILE_1.txt`` in ``./FOLDER_1``, which will be created if it does not exist:

.. code-block:: bash

 openstack object list CONTAINER_1

  +---------------------+
  | Name                |
  +---------------------+
  | FOLDER_1/FILE_1.txt |
  +---------------------+


.. warning::

  Local files will be overwritten if files with the same name are downloaded.


However, this command will fail if folders exist as unique objects in the container. For example, ``FOLDER_1/`` in the following:

.. code-block:: bash

 openstack object list CONTAINER_1

  +---------------------+
  | Name                |
  +---------------------+
  | FOLDER_1/           |
  | FOLDER_1/FILE_1.txt |
  +---------------------+

.. note::

  This occurs if folders have been created using the `Create Folder` screen, rather than the `Upload File` screen with the GUI, or if ``swift upload <container> <empty folder>`` has been used.
  The choice of folder creation mechanism should not affect the file structure when downloading containers/files or viewing files through the GUI.


In this case, the Swift client must be used to save containers:

.. code-block:: bash

  swift download CONTAINER_1


By default, this will save all files to the current directory, and, as before, any directories that do not exist will be created.


.. _swift_cli_delete_objects:

Deleting Files
--------------

Multiple objects can be deleted using:

.. code-block:: bash

  openstack object delete CONTAINER_1 FILE_1.txt FILE_2.txt


This will return nothing if successful.


.. _swift_cli_save_files:

Saving Files
------------

Individual files can be saved using the Openstack client. For example:

.. code-block:: bash

  openstack object save CONTAINER_1 FILE_1.txt


Multiple files can be saved using the Swift client. For example:

.. code-block:: bash

  swift download CONTAINER_1 FILE_1.txt FILE_2.txt


.. _swift_cli_save_folders:

Saving Folders
--------------

The full contents of a folder can be saved by using the Swift client to download each file. For example:

.. code-block:: bash

  swift download CONTAINER_1 FOLDER_1/FILE_1.txt FOLDER_1/FILE_2.txt


.. _swift_cli_delete_folders:

Deleting Folders
----------------

If a folder is not a unique object, but exists through file names, it can be deleted by deleting all files within the folder. For example, if ``FILE_1.txt`` and ``FILE_2.txt`` are the only files in ``FOLDER_1``, the following will delete the folder:

.. code-block:: bash

  openstack object delete CONTAINER_1 FOLDER_1/FILE_1.txt FOLDER_1/FILE_2.txt


If a folder is stored as a unique object, this can be deleted in the same way as a file:

.. code-block:: bash

  openstack object delete CONTAINER_1 FOLDER_1/


However, this will not delete any files within the folder. To delete the folder, both the folder object and the folder contents must be deleted:

.. code-block:: bash

  openstack object delete CONTAINER_1 FOLDER_1/ FOLDER_1/FILE_1.txt FOLDER_1/FILE_2.txt


.. _swift_cli_delete_containers:

Deleting Containers
-------------------

A container can be deleted using:

.. code-block:: bash

  openstack container delete CONTAINER_1


This will return nothing if successful. An error will be thrown if the container is not empty, unless the ``-r`` or ``--recursive`` options are used to delete all objects within the container at the same time.


.. _swift_cli_large_objects:

Large Files
-----------

Swift does not allow objects larger than 5GiB, so larger files must be segmented. This must be done using the Swift client:

.. code-block:: bash

  swift upload <container> <object> --segment-size <size>
  # <size> is the maximum segment size in Bytes. For example, to upload segments no larger than 1GiB:
  swift upload CONTAINER_1 FILE_1.txt --segment-size 1G


.. warning::

  Attempts to upload large files through the GUI or the Openstack client will fail.
  This may not occur until after an attempt has been made to upload the file, which may take a significant length of time.


This will upload the segments into a separate container, by default named <container>_segments, and create a "manifest" file describing the entire object in <container>.

.. note::

  A Dynamic Large Object is created by default, but if ``--use-slo`` is included with ``segment-size``, a Static Large Object will be created instead. This still allows concurrent upload of segments and downloads via a single object, but it does not rely on eventually consistent container listings.


The entire object can be downloaded via the manifest file as if it were any other file, through the GUI or using the Openstack client:

.. code-block:: bash

  openstack object save CONTAINER_1 FILE_1.txt


To delete the entire object, the Swift client must be used. For example:

.. code-block:: bash

  swift delete CONTAINER_1 FILE_1.txt


If successful, this will output the manifest file name, as well as the file names of each segment, all of which will have been deleted.
The segments container must be deleted separately.

.. warning::

  Attempting to delete a segmented file using ``openstack object delete`` will delete the manifest file, but not the segments.
  In this case, the folder containing the segments must be deleted manually, as described in the first example of :ref:`swift_cli_delete_folders`.


.. _swift_cli_copy_files:

Copying Files
-------------

Multiple files can be copied within a container, or between containers, using the Swift client.
For example, copying ``FILE_1.txt`` and ``FILE_2.txt`` from ``CONTAINER_1`` to ``CONTAINER_2``:

.. code-block:: bash

  swift copy --destination /CONTAINER_2 CONTAINER_1 FILE_1.txt FILE_2.txt


.. warning::

  This will overwrite any destination files sharing the same name.


If successful, this will create any containers and folders specified that do not exist, and output ``<file> copied to <destination>`` for each file.

.. note::

  The output will also include `created container <container>`, even for containers that already exist.


.. _swift_cli_editing_containers:

Editing Container Metadata
--------------------------

Multiple custom properties of a container can be added or overwritten simultaneously through repeated use of the ``--property`` option:

.. code-block:: bash

  openstack container set --property KEY_1=VALUE_1 --property KEY_2=VALUE_2 CONTAINER_1


.. note::

  Underscores ('_') in the property key will be converted to dashes ('-').


These properties will be listed when printing information about the container:

.. code-block:: bash

  openstack container show CONTAINER_1

  +--------------+----------------------------------+
  | Field        | Value                            |
  +--------------+----------------------------------+
  | account      | v1                               |
  | bytes_used   | 8                                |
  | container    | CONTAINER_1                      |
  | object_count | 3                                |
  | properties   | Key-1='VALUE_1', Key-2='VALUE_2' |
  +--------------+----------------------------------+


Custom properties may also be removed from a container:

.. code-block:: bash

  openstack container unset --property KEY_1 --property KEY_2 CONTAINER_1


Setting system metadata, such making a container public, can be done through the Swift client using:

.. code-block:: bash

  swift post <container> --read-acl ".r:*,.rlistings"


.. warning::

  The above command will allow the contents of a container to be viewed by anyone, with no authentication required.


  Access can instead be shared with specific groups. For example: ``<project-id>:<user-id>``, ``<project-id>:*``, ``*:<user-id>`` ``*:*`` or ``<role_name>``.
  User IDs, rather than names, should be used, as names are not globally unique.


Similarly, containers can be made private using:

.. code-block:: bash

  swift post <container> --read-acl ""


.. note::
  Container ACLs are stored in the `X-Container-Write` and `X-Container-Read` metadata, but are set by ``--write-acl`` and ``--read-acl`` respectively.

  Write access grants the ability to perform PUT, POST and DELETE operations on objects within a container, but not POST or DELETE operations on the container itself.

  Read access grants the ability to perform GET and HEAD operations on objects within a container, but access to privileged metadata such as `X-Container-Sync-Key` is not granted.


.. _swift_cli_editing_objects:

Editing Object Metadata
-----------------------

As for containers, multiple custom properties for object may be set simultaneously:

.. code-block:: bash

  openstack object set --property KEY_1=VALUE_1 --property KEY_2=VALUE_2 CONTAINER_1 FILE_1.txt


.. warning::

  Any existing object properties that are not listed will be removed, including the default ``Orig_Filename`` property.


.. note::

  Underscores ('_') in the property key will be converted to dashes ('-').


These properties will be listed when printing information about the object:

.. code-block:: bash

  openstack object show CONTAINER_1 FILE_1.txt

  +----------------+----------------------------------+
  | Field          | Value                            |
  +----------------+----------------------------------+
  | account        | v1                               |
  | container      | CONTAINER_1                      |
  | content-length | 4                                |
  | content-type   | text/plain                       |
  | etag           | 0cbc6611f5540bd0809a388dc95a615b |
  | last-modified  | Tue, 17 May 2022 17:35:29 GMT    |
  | object         | FILE_1.txt                       |
  | properties     | Key-1='VALUE_1', Key-2='VALUE_2' |
  +----------------+----------------------------------+


Custom properties may also be removed from objects:

.. code-block:: bash

  openstack object unset --property KEY_1 --property KEY_2 CONTAINER_1 FILE_1.txt


.. _swift_cli_tempurl:

Creating a Temporary URL
------------------------

Secret keys used in the cryptographic signature for temporary URLs can be created using the Swift client. For example, to set the secret key to ``MYKEY``:

.. code-block:: bash

  swift post -m "Temp-URL-Key:MYKEY"


.. note::

  Two secret key values per account, and two per container can be stored.


A temporary URL for a Swift object can then be generated using the Swift client, typically to allow GET or PUT access. For example:

.. code-block:: bash

  swift tempurl GET 1000 https://s3.echo.stfc.ac.uk/swift/v1/CONTAINER_1/FILE_1.txt MYKEY

  # This should return a URL similar to:
  https://s3.echo.stfc.ac.uk/swift/v1/CONTAINER_1/FILE_1.txt?temp_url_sig=?temp_url_sig=da39a3ee5e6b4b0d3255bfef95601890afd80709&temp_url_expires=1323479485


.. note::

  The URL returned includes an ampersand, so must be enclosed in quotation marks in a command shell.


References
----------

https://docs.openstack.org/python-openstackclient/train/cli/command-objects/container.html?

https://docs.openstack.org/python-openstackclient/train/cli/command-objects/object.html

https://docs.openstack.org/python-openstackclient/train/cli/decoder.html#swift-cli

https://docs.openstack.org/python-swiftclient/train/cli/index.html

https://docs.openstack.org/swift/train/overview_large_objects.html

https://docs.openstack.org/swift/train/overview_acl.html

https://docs.openstack.org/swift/train/api/temporary_url_middleware.html#secret-keys
