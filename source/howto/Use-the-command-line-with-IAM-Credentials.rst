======================================================================
How to use IAM Credentials on the command line
======================================================================

To use IAM credentials on the command line you first need to generate an application credential.

To do this log in to OpenStack. Expand Identity and click on Application Credentials as shown below

.. image:: /assets/howtos/IAM-Command-Line/Application-credential-1.png

Then Click on create application credentials

.. image:: /assets/howtos/IAM-Command-Line/Application-credential-2.png

Fill in the details here as required. We recommend setting the expiration date of the credential for no more than 1 month from the date you create the credential (if you have need of longer credentials contact the cloud team). 

If you need to make use of heat or create magnum clusters then tick the Unrestricted box. This allows the application credential to create a delegated user scope which is used by heat and magnum to perform actions on the user's behalf.

Then hit Create Application Credential.

.. image:: /assets/howtos/IAM-Command-Line/Application-credential-3.png

After a little thinking time you should see the following screen (secret redacted in the screen shot). From here hit Download Openrc File.

.. image:: /assets/howtos/IAM-Command-Line/Application-credential-4.png

The file should look something like this:

.. code-block:: bash

  #!/usr/bin/env bash

  export OS_AUTH_TYPE=v3applicationcredential
  export OS_AUTH_URL=https://openstack.stfc.ac.uk:5000/v3
  export OS_IDENTITY_API_VERSION=3
  export OS_REGION_NAME="RegionOne"
  export OS_INTERFACE=public
  export OS_APPLICATION_CREDENTIAL_ID=9b679e25821046c8a5e580069f5f5d45
  export OS_APPLICATION_CREDENTIAL_SECRET=<redacted>

Where the credential ID and Secret match what is shown in the Your application credential screen.

You should then be able to source the openrc file and run openstack commands as below:

.. code-block:: bash

  source Downloads/app-cred-test-for-docs-openrc.sh
  openstack server list

