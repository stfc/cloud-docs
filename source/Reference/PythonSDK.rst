###########
Python SDK
###########

.. contents::

Introduction
############

This document provides a brief setup guide for advanced users who want to use the Python SDK. This document assumes a good understanding of Python.

.. _clouds_yaml:

Setting Up Clouds.yaml
######################

Background
==========

Clouds.yaml allows a user to provide their credentials for a project separate to their script. This is strongly recommended for a number of reasons:

- Prevents hard-coded credentials leaking in the script
- Allows sharing of scripts for support reasons and to colleagues
- Able to quickly switch between a testing and production project

Setup
=====

- Navigate to `Openstack <https://openstack.stfc.ac.uk/>`_
- Go to **API Access**
- On the right click the dropdown to download the *clouds.yaml* file, not the RC file
- Move this file to `~/.config/openstack` you may need to create the parent directory

Your YAML file will look similar to:

.. code-block:: yaml

    clouds:
      openstack:
        auth:
          auth_url: https://openstack.stfc.ac.uk:5000/v3
          username: ...

- Modify the project name (default `openstack`) to something descriptive
- Add your password under the auth directive

For the above example:

.. code-block:: yaml

    clouds:
      my_proj_name:
        auth:
          auth_url: https://openstack.stfc.ac.uk:5000/v3
          username: exampleUser
          password: examplePassword

Accessing Multiple Projects
============================

This process can be repeated if you require access to multiple projects.

- Switch project and download YAML for additional project
- Append the contents of this downloaded file under the `clouds` directive

Here is an example of a file with multiple projects:

.. code-block:: yaml

    clouds:
      my_proj_name:
        auth:
          auth_url: https://openstack.stfc.ac.uk:5000/v3
          username: ...
          password: ...
      proj_dev:
        auth:
          auth_url: https://openstack.stfc.ac.uk:5000/v3
          username: ...
          password: ...
      proj_prod:
        auth:
          auth_url: https://openstack.stfc.ac.uk:5000/v3
          username: ...
          password: ...

This provides targets called `proj_one`, `proj_dev` and `proj_prod`. This is especially useful for testing a script against a development environment and performing a one-line change to switch to production.

Auth in Python Script
=====================

Openstack will automatically go to `~/.config/openstack/` to check for a clouds.yaml file (note: lower-case).

Using the above examples, we can easily point to the project named `my_proj_name` like so:

.. code-block:: python

    import openstack
    
    conn = openstack.connect(cloud='my_proj_name', region_name='RegionOne')

    # Print the list of servers to prove were connected as expected
    s = conn.compute.servers()
    print(s)

Using the Openstack SDK
#######################

.. important::

    Because the Openstack SDK dynamically populates the connection object with available methods most IDE type suggestions will be internal low-level methods and should be ignored. 
    
    For example `set_metadata` is always available instead of `set_x_metadata`, where x is a Openstack service.

The Openstack SDK dynamically populates the connection object at runtime. This is because Openstack fundamentally is a modular install with different plugins enabled by operators.

Instead Openstack adds "Proxies" to a connection object to represent what is available on the service. The list of proxies can be printed at runtime with:

.. code-block:: python

    import openstack

    connection = openstack.connect(cloud='my_proj_name', region_name='RegionOne')
    print(dir(connection))

Documentation
=============

The full list of `proxies can be viewed here <https://docs.openstack.org/openstacksdk/latest/user/#service-proxies>`_.

Worked Example - Set Server Metadata
------------------------------------

The `compute proxy <https://docs.openstack.org/openstacksdk/latest/user/proxies/compute.html>`_ allows us to manipulate compute resources (i.e. instances). 

Looking at the documentation linked above there are methods to get a list of server instances and a method to set metadata. If we want to set an arbitrary field of `FOO=BAR` on all servers we can do so using the compute module:

.. code-block:: python

    import openstack

    connection = openstack.connect(cloud='my_proj_name', region_name='RegionOne')
    server_list = connection.compute.servers()
    for server in server_list:
        connection.compute.set_server_metadata(server, FOO='BAR')
