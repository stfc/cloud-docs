.. _docker_nginx:

==================================================================
Running an NGinX webserver inside a Docker container
==================================================================

This tutorial shows how to run a webserver with docker.

Firstly create a VM - the instructions here are based on our ubuntu-focal-20.04-nogui image.

SSH to your VM and install docker:

.. code-block:: bash

  sudo apt-get update && sudo apt-get install docker.io

Start and enable docker

.. code-block:: bash

  sudo systemctl enable docker && sudo systemctl start docker

Add current user to the docker group **(Please do NOT run docker and containers as root)**

.. code-block:: bash

  sudo usermod -aG docker ${USER}

Or if you need to add a user that is not logged in

.. code-block:: bash

  sudo usermod -aG docker username

To apply new group you must logout and log back in 

.. code-block:: bash

  exit
  ssh user@<your_ip>

You can confirm changes have applied by running

.. code-block:: bash

  groups
  #Output:
  wheel docker

Pull the docker image for the nginx webserver

.. code-block:: bash

    docker pull nginx

Run the nginx docker container:

.. code-block:: bash

  docker run -it --rm -d -p 80:80 --name nginx nginx

Check that the container is running

.. code-block:: bash

  docker ps

You should see the nginx container running

You should now be able to browse to your webserver at the http://<instance-ip>

If this doesnt work check the security group for your instance, enabling http traffic through.
