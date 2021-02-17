.. _docker_mirror_guide:

STFC Docker Hub Mirror
***********************

Introduction
============

The STFC cloud team provides an internal Docker Hub mirror at https://dockerhub.stfc.ac.uk which is strongly recommended; users avoid rate limit issues, can pull large images faster over the local network and is free.

Docker Hub limit an IP to 100 pulls per 6 hours. Instances with floating IPs will have separate rate limits, whilst all other instances use a shared limit (as all pulls come from a single external IP).

Docker will automatically fall back to using Docker Hub directly if the mirror is ever unavailable, so there is minimal risk to applying these changes.

New Instances
=============

By default all new STFC Cloud instances will use the mirror by default. To confirm the mirror is being used see

Existing Instances
==================

Existing instances will automatically update to pull in the required file described here :ref:`restart_docker_service`. A restart of the instance or docker service is required to start using the mirror.

.. _check_docker_hub_mirror:

Checking that the mirror is being used
======================================

Checking configuration exists
------------------------------

Users can quickly check if they are using the mirror by running:

.. code :: console

    cat /etc/docker/daemon.json

An output similar to this indicates we are using the mirror:

.. code:: json

    {
        "registry-mirrors": ["https://dockerhub.stfc.ac.uk"]
    }

.. _docker_mirror_logs:

Monitoring Logs
---------------

For further peace of mind the logs can be checked. Docker will not produce any logs when pulling from the mirror successfully but will log an error if it fails to use direct connections:

- If this an existing instance which has not been restarted recently see :ref:`restart_docker_service`. 
- In a terminal follow the Docker logs:

.. code:: console

    # Where -u means unit and -f means follow
    journalctl -u docker -f 

- Select an image which has not been pulled on the System before, for example Ubuntu or Debian latest.
- In a separate terminal run a docker pull of your image:

.. code:: console

    # Assuming ubuntu:latest has never been pulled on the machine
    docker pull ubuntu:latest

- If the mirror could be contacted there will be no output in the logs about it
- If the mirror could not be used something similar to this will appear:

.. code:: text

    Jan 1 12:00:00 My-Instance-Name dockerd[900]: time="2021-01-01T12:00:00.000000000Z" level=info msg="Attempting next endpoint for pull after error: ...

For more support see :ref:`docker_mirror_troubleshooting`.

.. _manual_mirror_config:

Manually Configuring the Mirror
===============================

For most users this is not required; instances will automatically update the file as described in :ref:`check_docker_hub_mirror`.

If you have internal machines that are outside of Openstack or separately managed or you need to apply the changes proactively the following steps can be followed:

Docker Daemon
-------------
If you are using Docker (SL7 / Ubuntu / CoreOS / K8s < 1.20) the following steps can be performed using `sudo` or `root`. By default most distributions do not pre-create this file:

.. code:: console

    mkdir -p /etc/docker
    <editor> /etc/docker/daemon.json # e.g. vi /etc/docker/daemon.json

Add or append the following JSON:

.. code:: JSON

    {
        "registry-mirrors": ["https://dockerhub.stfc.ac.uk"]
    }

Then restart the service (see :ref:`restart_docker_service`).

Containerd
----------

As of Kubernetes 1.20, a future release (TBA) will use Containerd by default. The STFC Core OS image already contains the mirror information on users behalf at `/etc/containers/registries.conf`

Further documentation on manually on setting up containerd will be included when upstream Kubernetes uses containerd by default, as further internal testing is completed.

.. _restart_docker_service:

Restarting Docker
=================

.. Warning::

    Restarting the Docker Daemon will also pause and resume any running containers. This may result in service interruption or lost work.

The Docker Daemon can be restarted to apply the changes. Any running containers will be either paused and resumed, or restarted during this process.

Depending on the operating system the service is either called `docker` or `dockerd`, with the former being more common. To restart simply run:

.. code :: console
    
    systemctl restart docker  # or dockerd

To verify the service has resumed successfully:

.. code:: console
    
    systemctl status docker  # or dockerd

.. _docker_mirror_troubleshooting:

Troubleshooting Mirror Connection
=================================

The logs can be checked if you suspect the mirror is not being used (see :ref:`docker_mirror_logs`).

If Docker is failing back some simple diagnostics steps can be performed and/or cloud support can be contacted. Including the output of these steps in your email will enable us to provide support faster:

- Check that the VM has a connection to the internet with `ping example.com`
- Check config matches :ref:`manual_mirror_config`, paying attention to typos in the URL
- Check that you can connect to the mirror using curl: `curl https://dockerhub.stfc.ac.uk/v2/` you should get an empty response: {}
- Check the systemd logs for additional errors related to Docker `journalctl -u docker`
- Contact us for additional support
