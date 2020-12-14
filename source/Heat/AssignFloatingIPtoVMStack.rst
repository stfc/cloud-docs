==========================================
Assign Floating IPs to VMs in a Heat Stack
==========================================

Floating IPs can be assigned to VMs that are on private networks in order for them to be accessible
from an external network.

  This document assumes that Floating IPs have been assigned to your project. If you do not have any floating IPs in your project, please contact the cloud team.

There are two similar methods for assigning a floating IP to a virtual machine in a heat stack.
  1. Use the resource ``OS::Neutron::FloatingIPAssociation``.
  2. Add the ID of the floating IP in the networks property of the ``OS::Nova::Server`` Resource

For both methods, ``OS::Neutron::Port`` is used in order to define a port on the VM on which to attach the
floating IP.

To get the ID of the floating IPs, use the command:

.. code-block:: bash

  openstack floating ip list

This will return a list of floating IPs containing the IP address, pool, port, and ID.

Using OS::Neutron::FloatingIPAssociation
########################################

.. code-block:: yaml

  heat_template_version: <template-version>

  parameters:
    <define parameters here>

  resources:
    private_network_port: # define the VM port which will be used to attach the floating IP
      type: OS::Neutron::Port
      properties:
        network_id: <private network ID> #private network ID
        security_groups: [{get_param: security_group_id}]

    test_VM:
      type: OS::Nova::Server
      properties:
        image: {get_param: image_id}
        flavor: {get_param: flavor_id}
        key_name: {get_param: key_name}
        networks:
          - port: {get_resource: private_network_port}

    floating_ip_association:
      type: OS::Neutron::FloatingIPAssociation
      properties:
        floatingip_id: <ID of Floating IP address> #ID of IP address to assign to VM
        port_id: {get_resource: private_network_port} #port to attach floating IP


Using the Network Property in OS::Nova::Server
##############################################

.. code-block:: yaml

  heat_template_version: <temlate-version>

  parameters:
    <define parameters>

  resources:
    private_network_port:
      type: OS::Neutron::Port
      properties:
        network_id: <private-network-id> #private network ID
        security_groups: [{get_param: security_group_id}]

    test_VM:
      type: OS::Nova::Server
      properties:
        image: {get_param: image_id}
        flavor: {get_param: flavor_id}
        key_name: {get_param: key_name}
        networks: [{network: <private-network-id>, port: {get_resource: private_network_port}, floating_ip: <floating-ip-id> }]
