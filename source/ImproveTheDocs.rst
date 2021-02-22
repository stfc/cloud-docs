============================
Improving this Documentation
============================

You can help us to improve our documentation in several ways:

You can find the repository for this documentation at https://github.com/stfc/cloud-docs

If you have anything you would like to see documented then you can raise issues against the documentation and we will do our best to fill in the documentation.

Alternatively, you can email cloud support and we can open an issue on your behalf.

Any pull requests to improve or change the documentation, however minor, is welcome and greatly appreciated.

Setting Up a local development environment
------------------------------------------

The following assumes you are using Windows Subsystem for Linux with Ubuntu 20.04 LTS with GitHub Desktop to manage interaction with GitHub.

Create a fork on GitHub and clone it into GitHub desktop.

First install the software necessary to build the repository

.. code-block:: bash

  sudo apt-get update
  sudo apt-get install build-essential python3-sphinx python3-sphinx-rtd-theme

Change into the directory you cloned the repository to.

Run the following to build a local copy of the documentation:

.. code-block:: bash

  make html

Assuming that this succeeds, you can then open ./build/html/index.html in your web browser

Make any changes you wish to and then build again and test.

A good resource for editing rst is here: https://github.com/ralsina/rst-cheatsheet/blob/master/rst-cheatsheet.rst

Once you are happy with your changes, commit the changes and publish and then create a pull request for us to review.
