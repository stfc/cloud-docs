============================
Prevent Automatic Updates
============================

It is also possible to prevent our curated images from automatically updating certain packages or completely.

Preventing Automatic Updates of some packages
---------------------------------------------
It is possible to version lock certain packages you care about by changing the owner of /etc/yum/pluginconf.d/versionlock.list:

.. code-block:: bash

  sudo chown your-username /etc/yum/pluginconf.d/versionlock.list

Then add any packages to the above file in the same format as the other entries.

Preventing Automatic Updates completely
---------------------------------------
This is not recommended must only be done with the express written consent of the Cloud team. This can be obtained by emailing cloud-support@gridpp.rl.ac.uk

You still must comply with our terms of service.

Auto updating can be disabled by creating a file at /etc/noquattor. This file should contain the date the file is created and who created it.
