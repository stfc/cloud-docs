======================================================================
Forcibly update Scientific Linux and Centos Virtual Machines
======================================================================

When using our provided Scientific Linux or Centos images you may occasionally find that you get an error that a repository cannot be found when installing a package.

This is because we use snapshotted repositories for many of the common software repositories in order to minimise the risk of incompatible or broken packages. We periodically validate these snapshots and release them to our users and delete snapshots which are over 1 year old.

Our configuration management system periodically updates the configuration of these within your VMs.

If this hasn't happened it may be necessary to run these components manually. This process also maintains the monitoring and access for the Cloud Team

You can do that with the following command:

.. code-block:: bash

    quattor-fetch && quattor-configure --all

You should then see an output ending with this line:

.. code-block:: bash

    [OK]   0 errors, 0 warnings executing configure

Warning are usually fine and transient but if there are any errors or if this doesn't resolve your initial issue then please contact the Cloud team.
