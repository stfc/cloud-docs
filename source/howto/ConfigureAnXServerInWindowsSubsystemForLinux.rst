==============
Configuring an X Server in Windows Subsystem for Linux
==============

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
