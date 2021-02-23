.. _docker_nginx:

==================================================================
Running an NGinX webserver inside a Docker container
==================================================================

This tutorial shows how to run a webserver with docker.

Firstly create a VM - the instructions here are based on our ScientificLinux-7-NoGui image.

SSH to your VM and install docker:

.. code-block:: bash

  sudo yum install docker-ce -y

Start and enable dockerd

.. code-block:: bash

  sudo systemctl enable docker && sudo systemctl start docker

Check that you are using our dockerhub mirror as described here: :ref:`check_docker_hub_mirror`.

Pull the docker image for the nginx webserver

.. code-block:: bash

    sudo docker pull nginx

Run the nginx docker container:

.. code-block:: bash

  sudo docker run -it --rm -d -p 80:80 --name nginx nginx

Check that the container is running

.. code-block:: bash

  sudo docker ps

You should see the nginx container running

You should now be able to browse to your webserver at the http://<instance-ip>

If this doesnt work check the security group for your instance
