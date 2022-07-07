============
Image Types
============

We currently provide several curated images for use with the STFC Cloud.

These are images which have been preconfigured to meet our :ref:`tos` by having automatic updates enabled and reporting in to various monitoring systems.

We currently provide 2 operating systems Scientific Linux (a RedHat derivative) and Ubuntu

We aim to produce new versions of each image at least monthly with the previous version being renamed and deactivated.

The Images we provide are:

- ScientificLinux-7-AQ (scientificlinux-7-aq)
- ScientificLinux-7-Gui (scientificlinux-7-gui)
- ScientificLinux-7-NoGui (scientificlinux-7-nogui)
- ubuntu-focal-20.04-gui
- ubuntu-focal-20.04-nogui
- Ubuntu-Bionic-Gui (ubuntu-bionic-18.04-gui)
- Ubuntu-Bionic-NoGui (ubuntu-bionic-18.04-nogui)
- fedora-coreos-latest ( a container first OS used for Magnum cluster )

###############
Image Variants
###############
We provide 3 different variants of images:

- Gui - Includes a preinstalled desktop environment
- NoGui - A minimal install
- AQ - A minimal install bootstrapped into the Aquilon configuration management system (Only available to Aquilon users within STFC)

############################
Additional Operating systems
############################
We currently only offer the above operating systems as there is an overhead to providing the curated images although we are currently working on the provision of CentOS images.

We do allow users to upload their own images although it is then the users' responsibility to ensure any VMs created with these comply with our :ref:`tos`. Any non compliant VMs will be deleted.

Please contact us at cloud-support@stfc.ac.uk if you would like to upload your own images and discuss how to comply with the :ref:`tos`
