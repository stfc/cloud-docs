=====================
Creating a LAMP Stack
=====================

LAMP stacks contain a group of software consisting of:

  - Linux OS
  - Apache web server
  - mySQL Database
  - PHP

#########
Templates
#########

The following templates shows how an instance can be configured using a bash script
to install Apache, mySQL, and PHP. The bash script is executed during the VM's cloud-init phase.

**Note:** During the cloud-init phase, root executes any user-scripts.

> During the execution of the scripts, we will be 'holding' some of the packages to stop them being upgraded
during the `apt-get upgrade` step. This is because the upgrade requires a user response for the packages.
These can be upgraded by the user after VM configuration is complete by running the upgrade command.


Using OS::Heat::SoftwareConfig
##############################

The heat resource OS::Heat::SoftwareConfig has been used to define the user script.

OS::Heat::SoftwareConfig

.. code-block:: yaml

  <To add code here>

.. code-block:: yaml

  heat_template_version: 2018-03-02

  parameters:
    key_name:
      type: string
      default: <key-pair-name>
      description: Key Pair to use in order to SSH into the instance.
    image_id:
      type: string
      default: <image-id>
      description: The image for the instance
    flavor_id:
      type: string
      default: <flavor-id>
      description: The flavor for the instance
    security_group_id:
      type: string
      default: <security-group-id>

  resources:
    test_script:
      type: OS::Heat::SoftwareConfig
      properties:
        group: ungrouped
        config: |
          #!/bin/bash -v
          apt-get update
          apt-mark hold libpam-krb5 libpam-modules libpam-modules-bin libpam-runtime libpam-systemd libpam0g
          apt-get -y upgrade
          apt-mark unhold libpam-krb5 libpam-modules libpam-modules-bin libpam-runtime libpam-systemd libpam0g
          # mysql-server is installed at this stage (can install after VM creation)
          apt-get install -y apache2 php libapache2-mod-php php-mysql php-gd mysql-server

    test_VM:
      type: OS::Nova::Server
      properties:
        image: {get_param: image_id}
        flavor: {get_param: flavor_id}
        key_name: {get_param: key_name}
        networks:
          - network: Internal
        security_groups:
          - {get_param: security_group_id}
        user_data_format: SOFTWARE_CONFIG
        user_data: {get_resource: test_script}


Using OS::Nova::Server
########################
The bash script can be placed alternatively in the user_data property of the OS::Nova::Server resource.

.. code-block:: yaml
  heat_template_version: <template-version>

  parameters:
    <define parameters>

  resources:
    VM:
      type: OS::Nova::Server
      properties:
        image: {get_param: image_id}
        flavor: {get_param: flavor_id}
        key_name: {get_param: key_name}
        networks:
          - network: <network-name>
        security_groups:
          - {get_param: security_group_id}
        user_data: |
          #!/bin/bash -v
          apt-get update
          apt-mark hold libpam-krb5 libpam-modules libpam-modules-bin libpam-runtime libpam-systemd libpam0g
          apt-get -y upgrade
          apt-mark unhold libpam-krb5 libpam-modules libpam-modules-bin libpam-runtime libpam-systemd libpam0g
          # mysql-server is installed at this stage (can install after VM creation)
          apt-get install -y apache2 php libapache2-mod-php php-mysql php-gd mysql-server

When a Bash Script becomes too long or complex, the get_file function can be used to
retrieve and execute the bash script:

.. code-block:: yaml
  heat_template_version: <template-version>

  parameters:
    <define parameters>

  resources:
    VM:
      type: OS::Nova::Server
      properties:
        image: {get_param: image_id}
        flavor: {get_param: flavor_id}
        key_name: {get_param: key_name}
        networks:
          - network: <network-name>
        security_groups:
          - {get_param: security_group_id}
        user_data_format: RAW
        user_data: {get_file: bash-script.sh}

> **Note:** user_data_format is required for defining how the user_data should be formatted for the server.
 without this parameter when defining a bash script this way, cloud init returns errors in the log and cannot run the script.

The function str_replace can be used in order to set variable values in the bash script based on parameters or resources in the stack.

Example from Openstack: https://docs.openstack.org/heat/rocky/template_guide/software_deployment.html

.. code-block:: yaml
  heat_template_version: <template-version>

  parameters:
    <define parameters>

  resources:
    VM:
      type: OS::Nova::Server
      properties:
        image: {get_param: image_id}
        flavor: {get_param: flavor_id}
        key_name: {get_param: key_name}
        networks:
          - network: <network-name>
        security_groups:
          - {get_param: security_group_id}
        user_data:
          str_replace:
            template: |
          #!/bin/bash
          # ...
          params:
            $FOO: {get_param: foo}

> **Note:** If a stack-update is performed and any changes have been made to the stack update,
 then the server will be deleted and replaced.


> If you are using the above bash scripts to install mysql-server as well as the other components of the stack, it is best to also run mysql_secure_installation as well.
To automate mysql_secure_installation steps, please see the page **Installing and setting up MySQL database in a Stack**


References:
##########

https://docs.openstack.org/heat/rocky/template_guide/software_deployment.html

https://www.digitalocean.com/community/tutorials/how-to-install-linux-apache-mysql-php-lamp-stack-on-ubuntu-16-04
