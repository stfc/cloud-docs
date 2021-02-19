========================================================
Configuring an X Server in Windows Subsystem for Linux
========================================================

First select a Windows based X server such as Xming or VcXsrv

I chose to use VcXsrv, download and install this. At time of writing the default installation options work fine.
Launch VcXsrv by running XLaunch

Open your WSL window and run the following command to set your default display:

.. code-block:: bash

    export DISPLAY=localhost:0.0

You should then be able run X applications

You can test this by running

.. code-block:: bash

    sudo apt install x11-apps

then

.. code-block:: bash

    xclock

You should then see a little graphical clock appear somewhere on your screen.

To be able to use this remotely you will need to generate an XAuthority file. This can be done with the following:

.. code-block:: bash

      xauth generate :0 . trusted

You should then be able to ssh into a remote host with the following command provided that the remote host is appropriately configured :

.. code-block:: bash

    ssh -X <user>@<your.remote.host>

