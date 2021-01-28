======================
Create A Server Group
======================

**This document is in progress**

Server groups can be used to ensure that instances are either placed on the same
hypervisor (affinity) or are placed on different hypervisors (anti-affinity).

There are four policies which can be applied to a server group:

  - affinity
  - soft-affinity
  - anti-affinity
  - soft-anti-affinity

Server groups can be implemented in a Heat template using the resource OS::Nova::ServerGroup.

Syntax
######

.. code-block:: yaml

    resource:
      server_group:
        type: OS::Nova:ServerGroup
        properties:
          name: string #Optional, - Server group name. Any updates cause replacement.
          policies: [string, string] #Optional, a list of string policies to apply.

Example
#######

.. code-block:: yaml

  resources:

    affinity_group:
      type: OS::Nova::ServerGroup
      properties:
        name: hosts on same compute nodes
        policies:
          - affinity

    my_instance:
      type: OS::Nova::Server
      properties:
        image: { get_param: image_id }
        flavor: { get_param: flavor_id }
        key_name: {get_param: KeyName }
        networks:
          - network: Internal #define the network to use as internal
        security_groups:
          - { get_param: security_group_id }
        user_data_format: RAW
        name: server_1 #name for instance
          scheduler_hints:
            group: {get_resource: affinity_group}

You can list the server groups which are in your project using the command:

.. code-block:: bash

  openstack server group list

This should return a table similar to this one:

.. code-block:: bash

  +--------------------------------------+---------------------------------+----------+
  | ID                                   | Name                            | Policies |
  +--------------------------------------+---------------------------------+----------+
  | 6c8030c0-1b33-4470-b26d-51b6cac17bb7 | hosts on same compute nodes     | affinity |
  +--------------------------------------+---------------------------------+----------+

You can also list the members of the server group using:

.. code-block:: bash

  openstack server show <server-group-name/id>

For example:

.. code-block:: bash

  openstack server group show 6c8030c0-1b33-4470-b26d-51b6cac17bb7

  +----------+--------------------------------------+
  | Field    | Value                                |
  +----------+--------------------------------------+
  | id       | 6c8030c0-1b33-4470-b26d-51b6cac17bb7 |
  | members  | 87663bdb-c597-4098-b09c-624ec9974572 |
  | name     | hosts on same compute nodes          |
  | policies | affinity                             |
  +----------+--------------------------------------+


References
##########

https://docs.openstack.org/heat/latest/template_guide/openstack.html#OS::Nova::ServerGroup

https://docs.syseleven.de/syseleven-stack/en/tutorials/affinity

https://docs.openstack.org/heat/latest/template_guide/openstack.html#OS::Nova::ServerGroup

https://docs.openstack.org/mitaka/config-reference/compute/scheduler.html#servergroupaffinityfilter
