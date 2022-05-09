Swift
=====

Swift is a highly available, distributed, eventually consistent object/blob store. Swift allows lots of data to be stored efficiently, safely, and cheaply. It's built for scale and optimized for durability, availability, and concurrency across the entire data set. Swift is ideal for storing unstructured data that can grow without bound.

Files and data are uploaded as **objects**, which are organised into **containers**.

.. note::

  Swift containers should not be confused with Docker containers. Containers may also be referred to as buckets or pools.


Although you cannot nest directories in object storage, a hierarchical structure can be simulated within a container through the inclusion of delimiters ('/') in object names.

Swift CLI
---------

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


Commands
~~~~~~~~

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

  Containers created using the CLI will not be publicly accessible. This can be changed via the GUI.


.. _Upload-Objects-CLI:


Uploading Objects
-----------------

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

Multiple files up me uploaded simultaneously by listing then after the container name:

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

  The name of the object uploaded will include its relative local path, unless otherwise specified. For example, if ./FOLDER_1/FILE_1.txt is uploaded, it will be named FOLDER_1/FILE_1.txt in the container by default.

Swift does not allow objects larger than 5GiB, so larger files must be segmented. Attempts to upload large files through the GUI or the Openstack client will fail, so the Swift client is required:

.. code-block:: bash

  swift upload <container> <object> --segment-size <size>
  # <size> is the maximum segment size in Bytes, e.g. to upload segments no larger than 1GiB:
  swift upload CONTAINER_1 FILE_1.txt --segment-size 1G

This will upload the segments into a separate container, by default named <container>_segments, and create a "manifest" file describing the entire object in <container>.

.. note::

  A Dynamic Large Object is created by default, but if ``--use-slo`` is included with ``segment-size``, a Static Large Object will be created instead. This still allows concurrent upload of segments and downloads via a single object, but it does not rely on eventually consistent container listings.

The entire object can be downloaded via the manifest file as if it were any other file, through the GUI or using the Openstack client:

.. code-block:: bash

  openstack object save CONTAINER_1 FILE_1.txt


References
----------

https://www.openstack.org/software/releases/train/components/swift

https://docs.openstack.org/swift/train/

https://docs.openstack.org/swift/train/api/pseudo-hierarchical-folders-directories.html

https://docs.openstack.org/python-openstackclient/train/cli/decoder.html#swift-cli

https://docs.openstack.org/python-swiftclient/train/cli/index.html

https://docs.openstack.org/swift/train/overview_large_objects.html