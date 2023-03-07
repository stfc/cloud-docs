.. _application_credentials:

=======================
Application Credentials
=======================

Application credentials can be used to allow applications or other users certain access to specific projects
without embedding the main user's password in the application configuration.

Application credentials can be created either through the web interface or command line

.. note::

  Here we assume that users are familiar with using ``clouds.yaml``. 
  See :ref:`clouds_yaml` for more information.


Create Credential from the Web interface
########################################

The option for generating application credentials can be found under *Identity -> Application Credentials*.
Select *Create Application Credential*.

The form has 7 fields:

- **Name:** The name that is given to the application credential.

- **Description:** Description of the application credential. This could be the description of which application will use this credential or the purpose of the credential.

- **Secret:** A password for the credential which will be used in order to authenticate access through keystone. We recommend you leave this blank or use a tool like *pwgen* to generate this automatically

- **Expiration Date:** Date that the application credential expires and will be deleted. If left blank, the application credential will not expire.

- **Expiration Time:** The time the credential expires, if a date is given by default the time is 00:00:00.

- **Roles**: The role that the application credential is authorized to have.

- **Unrestricted Option:** By default this is left unticked.

.. warning:: 

    If ticked, the application credential will be able to create additional application credentials.

    **For security, always have the application credential restricted.**
    Restrictions on these operations are deliberately imposed as a safeguard to prevent a compromised application credential from regenerating itself.

It is important to make a note of the secret which has been chosen for the application credential as it is only revealed once after the application credential has been generated.
From the web interface, the RC file and the ``cloud.yaml`` file is available once and it is recommended to download these files.
If the user has forgotten the secret, a new application credential will have to be created.

Create Credential from Command Line
###################################

To view the list of application credentials, we can use the command:

.. code-block:: bash

  openstack application credential list

This will return an empty line if no application credentials have been created yet,
or a table similar to the one below:

.. code-block:: bash

  +------------------------------+-----------------------+----------------------+-------------------------------- ------------------------------------------+----------------------------+
  | ID                           | Name                  | Project ID           | Description                                                               | Expires At                 |
  +------------------------------+-----------------------+----------------------+---------------------------------------------------------------------------+----------------------------+
  | APPLICATION_CREDENTIAL_ID_1  | Test-App-Credential-1 | PROJECT_ID           | This is  a test application credential generated using the web interface. | 2020-07-01T00:00:00.000000 |
  | APPLICATION_CREDENTIAL_ID_2  | Test-app-credential-2 | PROJECT_ID           | Test Application Credential from command line.                            | None                       |
  +------------------------------+-----------------------+----------------------+---------------------------------------------------------------------------+----------------------------+

To create a new application credential, you can use the command:

.. code-block:: bash

  openstack application credential create [-h]
                                               [-f {json,shell,table,value,yaml}]
                                               [-c COLUMN] [--noindent]
                                               [--prefix PREFIX]
                                               [--max-width <integer>]
                                               [--fit-width] [--print-empty]
                                               [--secret <secret>]
                                               [--role <role>]
                                               [--expiration <expiration>]
                                               [--description <description>]
                                               [--unrestricted] [-:-restricted]
                                               <name>

Below is an example of creating an application credential which expires on 06/07/2020 at 00:00:00 and the secret has been generated automatically by OpenStack.


.. code-block:: bash

  openstack application credential create --expiration 2020-07-08T00:00:00 --description "Example Application Credential" Example-Credential

  +--------------+----------------------------------------------------------------------------------------+
  | Field        | Value                                                                                  |
  +--------------+----------------------------------------------------------------------------------------+
  | description  | Example Application Credential                                                         |
  | expires_at   | 2020-07-08T00:00:00.000000                                                             |
  | id           | APPLICATION_CREDENTIAL_ID                                                              |
  | name         | Example-Credential                                                                     |
  | project_id   | PROJECT_ID                                                                             |
  | roles        | user                                                                                   |
  | secret       | SECRET                                                                                 |
  | system       | None                                                                                   |
  | unrestricted | False                                                                                  |
  | user_id      | USER_ID                                                                                |
  +--------------+----------------------------------------------------------------------------------------+

.. note:: 

    The secret is only revealed **once**. If a user has forgotten the secret, a new application credential as to be generated.

After an application credential has expired, it is still visible in the application credential list.
If the application credential is used after it has expired, nothing will happen and no one can get access to the project via the expired credential.

RC source and clouds.yaml file
##############################

Unlike in the web interface, the RC file and the clouds.yaml file are not automatically generated. They need to be created separately by the user.
The following are examples of a clouds.yaml file and RC file for an application credential.

clouds.yaml
-----------

.. code-block:: yaml

  # This is a clouds.yaml file, which can be used by OpenStack tools as a source
  # of configuration on how to connect to a cloud. If this is your only cloud,
  # just put this file in ~/.config/openstack/clouds.yaml and tools like
  # python-openstackclient will just work with no further config. (You will need
  # to add your password to the auth section)
  # If you have more than one cloud account, add the cloud entry to the clouds
  # section of your existing file and you can refer to them by name with
  # OS_CLOUD=openstack or --os-cloud=openstack
  clouds:
    openstack:
      auth:
        auth_url: AUTH_URL
        application_credential_id: "APP_CREDENTAL_ID"
        application_credential_secret: "APP_CREDENTIAL_SECRET"
      region_name: "RegionOne"
      interface: "public"
      identity_api_version: 3
      auth_type: "v3applicationcredential"


RC File
-------

.. code-block:: bash

  #!/usr/bin/env bash
  export OS_AUTH_TYPE=v3applicationcredential
  export OS_AUTH_URL=https://AUTH-URL #this will be the Identity service endpoint URL under API Access
  export OS_IDENTITY_API_VERSION=3
  export OS_REGION_NAME="RegionOne"
  export OS_INTERFACE=public
  export OS_APPLICATION_CREDENTIAL_ID=APP_CREDENTIAL_ID
  export OS_APPLICATION_CREDENTIAL_SECRET=APP_CREDENTIAL_SECRET


References
###########

https://docs.openstack.org/keystone/latest/user/application_credentials.html
https://docs.openstack.org/api-ref/identity/v3/index.html?expanded=authenticating-with-an-application-credential-detail#application-credentials
https://cloud.garr.it/compute/app-credential/
https://access.redhat.com/documentation/en-us/red_hat_openstack_platform/14/html/users_and_identity_management_guide/application_credentials
