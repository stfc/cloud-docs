========================================
Ansible and terraform on STFC cloud
========================================

.. contents:: Table of Contents

Why Infrastructure as code (IaC)?
-------------------------------------

IaC allows users to avoid manual configuration of environments and enforce consistency by representing the desired state of their environments as code. They are repeatable and prevent runtime issues caused by configuration drift or missing dependencies.

Always treat your resources (VMs etc.) like cattle, not pets, they are all replaceable, and you shouldn't directly SSH into the machine and make specific changes. 

Tools for IaC
----------------------
We recommend two tools for this purpose. `Terraform` and `Ansible`.

Terraform V.S. Ansible
^^^^^^^^^^^^^^^^^^^^^^^^^^
Always choose the tool you need for the job, Terraform is usually used to create new infrastructure, and Ansible is used to configure and manage the infrastructure.

Terraform
^^^^^^^^^^^
Terraform is an infrastructure as code (IaC) tool for creating, maintaining and decommissioning large data center infrastructure. The configurations are specified in a declarative language, where it will then determine the steps to transform infrastructures to the new desired state.

Advantages:
* Plan: A useful feature of Terraform is the ``terraform plan`` where it will perform a dry run and show the changes required without actually performing them.
* It is independent of cloud provider.
* It can integrate multiple cloud providers. This allows a hybrid cloud approach that provision resources across multiple providers.

Installing Terraform
"""""""""""""""""""""""
On a Ubuntu machine run the following command

.. code-block:: bash

    # register key
    curl -fsSL https://apt.releases.hashicorp.com/gpg | sudo apt-key add -
    # add repo
    sudo apt-add-repository "deb [arch=$(dpkg --print-architecture)] https://apt.releases.hashicorp.com $(lsb_release -cs) main"
    # install with apt
    sudo apt install terraform

Setting up provider
"""""""""""""""""""""

1. Create a directory and ``cd`` to the directory
2. Create a file named ``providers.tf`` with the following content. This will enable the `OpenStack` provider.

.. code-block:: bash
    :caption: providers.tf

    # Define required providers
    terraform {
    required_version = ">= 0.14.0"
    required_providers {
        openstack = {
        source  = "terraform-provider-openstack/openstack"
        version = "~> 1.35.0"
        }
    }
    }
    
    provider "openstack" {}

3. Declare the cloud information by ``source <openstack-rc-file>``. 

.. note::
    
    See :ref:`Use-OpenStack-CLI` on login as using ``provider.tf`` is not secure

4. Run ``terraform init``

.. code-block:: bash

    $ terraform init
    
    Initializing the backend...
    
    Initializing provider plugins...
    - Reusing previous version of terraform-provider-openstack/openstack from the dependency lock file
    - Using previously-installed terraform-provider-openstack/openstack v1.35.0
    
    Terraform has been successfully initialized!
    
    You may now begin working with Terraform. Try running "terraform plan" to see
    any changes that are required for your infrastructure. All Terraform commands
    should now work.
    
    If you ever set or change modules or backend configuration for Terraform,
    rerun this command to reinitialize your working directory. If you forget, other
    commands will detect it and remind you to do so if necessary.

Applying config via terraform
"""""""""""""""""""""""""""""""
To create a configuration you need the ``deploy.tf`` for your configuration and ``variables.tf`` for the variables in the provider folder 

I have create an example that will create a set of instances with a volume attached to it.

.. code-block:: bash
    :caption: variables.tf

    # create a SSH key-pair using other method first since terraform is not secure
    variable "key_pair_name" {
        description = "key pair name"
        default = "tutorial"
    }
    
    variable "http_sec_group" {
        description = "custom security group name"
        default = "HTTP-ingress"
    }
    
    variable "network_name" {
        description = "The network to be used."
        default  = "Internal"
    }
    
    variable "instance_name" {
        description = "The Instance Name to be used."
        default  = "tutorial-machine"
    }
    
    variable "image_id" {
        #find with: openstack image list
        description = "The image ID to be used."
        default  = "622cb70d-c88c-4dc4-99e6-df8b8f9965d7"
    }
    
    variable "flavor_id" {
        #find with: openstack flavor list
        description = "The flavor id to be used."
        default  = "026ace2c-5247-4bdc-8929-81d129cc69bf"
    }
    
    # have to put http sec group here as well
    variable "security_groups" {
        description = "List of security group"
        type = list
        default = ["default", "HTTP-ingress"]
    }
    
    variable "instance_num" {
        description = "The keypair public key."
        default = 1
    }
    
    variable "volume_name" {
        description = "name of volume"
        default = "tutorial_volume"
    }
    
    variable "volume_size" {
        description = "size of volume"
        default = 3
    }

.. code-block:: bash
    :caption: delpoy.tf

    # create a security group named secgroup_1
    resource "openstack_networking_secgroup_v2" "secgroup_1" {
    # you can access the variables using var.<variable-name>
    name        = var.http_sec_group
    description = "My neutron security group"
    }
    
    # create a security group rules
    resource "openstack_networking_secgroup_rule_v2" "secgroup_rule_1" {
    direction         = "ingress"
    ethertype         = "IPv4"
    protocol          = "tcp"
    port_range_min    = 80
    port_range_max    = 80
    remote_ip_prefix  = "0.0.0.0/0"
    # you can access the output attribute of other resources using ${<provider>.<name>.<attribute>}
    security_group_id = "${openstack_networking_secgroup_v2.secgroup_1.id}"
    }
    
    #creating instances
    resource "openstack_compute_instance_v2" "Instance" {
    # the count will create a number of duplicates equal to the number
    count = var.instance_num
    name = format("%s-%02d", var.instance_name, count.index+1)
    image_id = var.image_id
    flavor_id = var.flavor_id
    key_pair = var.key_pair_name
    security_groups = var.security_groups
    network {
        name = var.network_name
    }
    }
    
    resource "openstack_blockstorage_volume_v2" "Volume" {
    count = var.instance_num
    name        = format("%s-%02d", var.volume_name, count.index+1)
    size        = var.volume_size
    }
    
    resource "openstack_compute_volume_attach_v2" "attachments" {
    count       = var.instance_num
    # if there are multiples of same resources the detail are stored in an array
    instance_id = "${openstack_compute_instance_v2.Instance[count.index].id}"
    volume_id   = "${openstack_blockstorage_volume_v2.Volume[count.index].id}"
    }
    
    # this will create a output which you can use
    output "instance_ips" {
        value = {
            for instance in openstack_compute_instance_v2.Instance:
            instance.name => instance.access_ip_v4
        }
    }

You should create ssh keys via the web interface or OpenStack clients and reference it in terraform since terraform will store configuration in plain text which will expose your ssh key. 

After you have edited the files, you can run ``terraform plan`` or ``terraform apply`` to review changes and apply them to the cloud.

To change the variable you can either change the ``default`` in ``variable.tf`` or by passing the ``-var "<variable-name>=<value>"``  flag during ``terraform plan`` or ``terraform apply``. 

Example:

.. code-block:: bash
    
    $ terraform plan -var "instance_num=2"


.. code-block:: bash
    
    $ terraform apply -var "instance_num=2"
    ...
    Apply complete! Resources: 8 added, 0 changed, 0 destroyed.
    
    Outputs:
    
    instance_ips = {
    "tutorial-machine-01" = "172.16.100.135"
    "tutorial-machine-02" = "172.16.101.124"
    }

Reference
"""""""""""""
You should refer to `Docs overview | terraform-provider-openstack/openstack | Terraform Registry <https://registry.terraform.io/providers/terraform-provider-openstack/openstack/latest/docs>`_ for details on how to use the providers



.. list-table:: Common providers
   :header-rows: 1

   * - Provider
     - Function
   * - ``openstack_compute_instance_v2``
     - Create instance
   * - ``openstack_networking_secgroup_v2``
     - Create security group
   * - ``openstack_networking_secgroup_rule_v2``
     - Create security group rule
   * - ``openstack_networking_network_v2``
     - Create network   
   * - ``openstack_networking_floatingip_v2``
     - Get a floating IP from allocated pool
   * - ``openstack_compute_floatingip_associate_v2``
     - Associate floating IP to instances
   * - ``openstack_containerinfra_clustertemplate_v1``
     - Create magnum cluster template
   * - ``openstack_containerinfra_cluster_v1``
     - Create magnum cluster    
   * - ``openstack_blockstorage_volume_v2``
     - Create a new volume
   * - ``openstack_compute_volume_attach_v2``
     - Attach volume to instances
   * - ``openstack_lb_loadbalancer_v2``
     - Create Load balancer
   * - ``openstack_lb_listener_v2``
     - Create Load balancer listener    
   * - ``openstack_lb_pool_v2``
     - Set method for load balance charge between instance (e.g. round robin)        
   * - ``openstack_lb_member_v2``
     - Add instance to load balancer    
   * - ``openstack_lb_monitor_v2``
     - Create health monitor for load balancer        

Ansible
^^^^^^^^^^^
Ansible is a Python-based IT system configuration automation tool that gained widespread use as a network automation system

Advantages:
* It uses YAML based playbook
* It is agentless, which means that you don't have to load anything to the target machine.
* Large community with open source and vendor support

Installing ansible
"""""""""""""""""""""

.. code-block:: bash

    sudo apt update
    sudo apt install software-properties-common
    sudo apt-add-repository ppa:ansible/ansible
    sudo apt update
    sudo apt install ansible

Setting up target
""""""""""""""""""""""""""
Create a Host file refer to this `How to build your inventory — Ansible Documentation <https://docs.ansible.com/ansible/latest/user_guide/intro_inventory.html>`_

.. code-block:: YAML
    :caption: hosts

    all:
    # group of remote
    hosts:
        172.16.100.135:
        172.16.101.124:

Preparing key file
""""""""""""""""""""""""""
You should use the ``*.pem`` file you downloaded when you created the key

Your ssh key must not be too open set permission using ``sudo chmod 600 <path-to-key>``

Setting up playbook
"""""""""""""""""""""""""
Here is an example of a simple playbook. you can refer to ansible documentations for different providers. 

.. code-block:: YAML
    :caption: hosts

    - hosts: all
    tasks:
        - name: Print message
        debug:
            msg: Hello Ansible World

Running ansible-playbook
""""""""""""""""""""""""""""
``ansible-playbook -i ./hosts --private-key <path-to-key> playbook.yml``

Example

.. code-block:: bash

    $ ansible-playbook -i ./hosts --private-key ./key.pem playbook.yml
    PLAY [all] *******************************************************************************************************************************************************************************************
    
    TASK [Gathering Facts] *******************************************************************************************************************************************************************************
    ok: [172.16.100.135]
    ok: [172.16.101.124]
    
    TASK [Print message] *********************************************************************************************************************************************************************************
    ok: [172.16.100.135] => {
        "msg": "Hello Ansible World"
    }
    ok: [172.16.101.124] => {
        "msg": "Hello Ansible World"
    }
    
    PLAY RECAP *******************************************************************************************************************************************************************************************
    172.16.100.135             : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
    172.16.101.124             : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0

Reference
"""""""""""""

`Ansible Documentation — Ansible Documentation <https://docs.ansible.com/ansible/latest/index.html>`_