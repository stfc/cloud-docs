======================================================================
Configuring SSH Agent Forwarding in Windows Subsystem for Linux
======================================================================

Configuring WSL is pretty easy, this link offers a good tutorial.

https://www.windowscentral.com/install-windows-subsystem-linux-windows-10

Ensure your distribution of Linux in WSL is fully up to date (some versions have known issues around agent forwarding).

I used Ubuntu 18.04 LTS so I ran:

.. code-block:: bash

    sudo apt-get update
    sudo apt-get upgrade

This left me with a nice up to date install.
You can then run the following commands to start and add your key to the ssh agent:

.. code-block:: bash

    eval $(ssh-agent -s)
    ssh-add \<path-to-your-ssh-private-key\>

Then you can ssh to your host with agent forwarding using

.. code-block:: bash

    ssh -A \<your-remote-host\>

Then you can SSH on from there.
