
==========================
Lifecycle of a Virtual Machine
==========================

Managing VMs is key to maintaining good security. Preparing a plan to patch, reboot and migrate to new OSes is the best way to ensure maintenance is carried out regularly.

The general attitude towards VMs should be that they are cattle, not pets - any services/workflows running on a VM should be easily replicable so that an individual host is not required to be kept around indefinitely. There are many ways to simplify setting up a VM, such as `Magnum clusters <https://stfc-cloud-docs.readthedocs.io/en/latest/Magnum/index.html>`_ and `Heat stacks <https://stfc-cloud-docs.readthedocs.io/en/latest/Heat/index.html>`_.

A good lifecycle management process also promotes good service management practices.

VM age
-------
VMs should not be left to age for too long for a few reasons, primarily security - vulnerabilities are more common in older machines, not all are patched. Also the hardware underneath and the flavor of the VM are not immortal - at some point they will no longer work/be supported.

We recommend getting rid of VMs at an age of 6 months if possible, and older than a year is usually not ideal. If your machines are approaching this age, consider migrating to newer flavors.

Steps to take
------------
As a user of the STFC Cloud, you are responsible for your machines, whilst the Cloud team is responsible fo the OS images (excluding custom images). Some steps you can take to keep your machines in good condition are:
 * Cycling out VMs on a schedule (ideally once every 6 months)
 * Update VMs regularly, and reboot them (many updates do not take effect until after a reboot)
 * Comply with security notifications from the Cloud team

Patching and rebooting VMs will extend their lifecycle by a certain amount, but eventually all will need to be replaced.

Making things easier
-------
 * Configuration management is your friend - can create machines with packages pre-installed
 * If you're running a service - design for high availability so that one machine going down for maintenance doesn't take the service down with it
 * For one-off VMs - use `Openstack Volumes <https://stfc-cloud-docs.readthedocs.io/en/latest/howto/Volume.html>`_ or other non-root-disk storage so that data isn't lost when cycling VMs
 * Stick to a reboot schedule

Finally, the Cloud team are here to help - submit a ticket if you have issues or use the Slack to ask questions.