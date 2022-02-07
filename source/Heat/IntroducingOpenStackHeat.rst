Introducing OpenStack Heat
###########################


OpenStack Heat is the Orchestration service for OpenStack and orchestrates infrastructure resources (such as VMs, floating IPs, volumes, networks etc.) for cloud applications.
This is done using templates in the form of YAML files which contain the properties and the relationships between resources.

Heat Templates allow relationships to be defined between difference resources, allowing Heat to call different OpenStack APIs to create different resources such as LBaaS Pools (Octavia), servers and server groups (Nova),
alarms (Aodh), volumes (Cinder), networks and security groups (Neutron). Heat manages the lifecycle of the stack and when a stack needs to be updated, an updated template can be applied to the existing stack. These templates can be integrated
with software configuration management tools such as Ansible and Puppet.

Architecture
-------------

Heat provides an AWS CloudFormation implementation for OpenStack. Heat provides integration of other core OpenStack components into a one-file template system. This system not only allows resources
from most OpenStack Projects to be created in a single template, it also provides more functionality including auto scaling of VMs and nested stacks.


**heat:** A CLI that communicates with the heat-api to execute AWS CloudFormation APIs. End developer could use heat REST API directly.

**heat-api:** provides an OpenStack-native REST API that processes API requests by sending them to the heat engine over RPC

**heat-api-cfn:** provides AWS Query API that is compatible with AWS CloudFormation and processes API requests by sending them to the heat-engine over RPC

**heat-engine:** orchestrates the launching of templates and provides events back to API consumer.


Heat Commands
--------------

To use Heat commands in the command line, we need to install the following package:

.. code-block:: bash

  pip install python-heatclient

To test that this has installed correctly, and that we are able to access the Heat API, we can run the command:

.. code-block:: bash

  openstack stack list

  #This should return an empty line if there are no stacks in the project or a table similar to the following:

  +---------------------------------+---------------------------------+-----------------+----------------------+----------------------+
  | ID                              | Stack Name                      | Stack Status    | Creation Time        | Updated Time         |
  +---------------------------------+---------------------------------+-----------------+----------------------+----------------------+
  | a00fa2cd-3e29-489f-8d9f-f805956 | software-deployment-test        | CREATE_COMPLETE | 2020-12-09T08:34:15Z | None                 |
  | 7d045                           |                                 |                 |                      |                      |
  | 1cb66fbd-1336-414b-b112-b0fcffe | spark-standalone-cluster        | CREATE_COMPLETE | 2020-12-07T10:31:58Z | None                 |
  | b0645                           |                                 |                 |                      |                      |
  | b7263b67-65c4-4333-abd0-7033afa | spark-stack-2                   | CREATE_FAILED   | 2020-12-04T11:42:25Z | None                 |
  | 961b6                           |                                 |                 |                      |                      |
  | c9c10097-c275-4c44-9324-0eb2f7a | docker-script-test              | CREATE_COMPLETE | 2020-12-02T16:39:54Z | None                 |
  | d60cf                           |                                 |                 |                      |                      |
  +---------------------------------+---------------------------------+-----------------+----------------------+----------------------+


The following list is the list of commands from Heat which can be used in OpenStack:


.. code-block:: bash

  # Commands provided by Heat are of the form:
  openstack stack <command> <options>

  stack abandon # abandon a stack and output results
  stack adopt # adopt a stack
  stack cancel # cancel create or update task for a stack
  stack check # check stack and its resources
  stack create # create a stack
  stack delete  # delete a stack
  stack environment show #
  stack event list # List stack events
  stack event show # View stack event
  stack export # export stack data json
  stack failures list # List failed resources in a stack
  stack file list # show a stack's files map
  stack hook clear #clear resource hooks on a given stack
  stack hook poll # list resources with pending hook for a stack
  stack list # list stacks in the project
  stack output list # list stack outputs
  stack output show # view stack output
  stack resource list # list stack resources
  stack resource mark unhealthy # mark one of the stack resources as unhealthy
  stack resource metadata # view metadata for stack's resource
  stack resource show # view details about a stack's resource
  stack resource signal # signal a resource with optional data (JSON data)
  stack resume # resume a stack
  stack show # view details about a stack
  stack snapshot create # create a snapshot of the stack
  stack snapshot delete # delete stack snapshot
  stack snapshot list # list stack snapshots
  stack snapshot restore # restore stack snapshot
  stack snapshot show # view details of a stack snapshot
  stack suspend # suspend a stack
  stack template show # view stack template
  stack update # update a stack using an updated template



Stacks
-------

**Stacks:** a collection of resources and their associated configuration

**Template:** A YAML file defining the resources which make up the stack. In OpenStack, templates follow the Heat Orchestration Template (HOT) format.

    **Note:** While Heat can interpret CFN (CloudFormation) Templates, they are _not_ backwards compatible with Heat Orchestrated Templates. It is recommended to use Heat Orchestrated Templates to create stacks.


Heat Orchestrated Templates
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Heat Orchestrated Templates are YAML files that instruct Heat which resources to create and the relationships between resources. Ansible has documentation on how to write YAML files that can be found here: https://docs.ansible.com/ansible/latest/reference_appendices/YAMLSyntax.html

Heat Templates consist of seven sections:

1. ``heat_template_version:`` (**required**) Indicates which format and features are used and supported when creating the stack.

2. ``description:`` (optional) Description of the stack template. It is recommended to include a description in templates to describe what users can do with the template.

3. ``parameter_groups:`` (optional) Defines how to group input parameters and the order of the parameters.

4. ``parameters:`` (optional) Defines input parameters. This section can be omitted if there are no input values required.

5. ``resources:`` (**required**) Defines resources in the template. At least one resource should be defined in a HOT template.

6. ``outputs:`` (optional) Defines output parameters available to users once the template has been instantiated.

7. ``conditions:`` (optional) Includes statements which can be used to apply conditions to a resource, for example a resource is created only when a property is defined or when another resource has been created first.


The structure of a HOT template is given as:

.. code-block:: yaml

  heat_template_version: 2018-08-31 #OpenStack Version we want to use.
                                    #Here, we want to use template for the Rocky release onwards

  description: #description of the template

  parameter_groups: #declares the parameter group and order.
    #This is not a compulsory section, however it is useful for grouping
    #parameters together when building more complex templates.

  parameters: #declares the parameters for resources

  resources: #declares the template resources
  # e.g. alarms, floating IPs, instances etc.

  outputs: #declares the output of the stack

  conditions: #declares any conditions on the stack


Please see the documentation **Create a Heat Stack** (https://stfc-cloud-docs.readthedocs.io/en/latest/howto/CreateAHeatStack.html) for an introduction to creating a stack using a HOT template.


Rocky Heat Templates
^^^^^^^^^^^^^^^^^^^^^

Templates which use:

.. code-block:: yaml

  heat_template_version: 2018-08-31

  #Or

  heat_template_version: rocky

Indicate that the template is a HOT template and has features added and/or removed up to the Queens Release. The list of supported functions in a Rocky Heat Template is:

.. code-block:: text

  digest    # allows for performing digest operations on a given value
  filter      # removes values from list
  get_attr    # references an attribute of a resource
  get_file    # returns the content of  a file into the template. Use to include files containing scripts or configuration files
  get_param   # references an input parameter of a template
  get_resource    # references another resource in the same template
  list_join   # joins a string with the given delimiter
  make_url    # builds URLs
  list_concat   # concatenates lists together
  list_concat_unique    # behaves identically to list_concat. Only removes repeating items of lists
  contains    # checks whether a specific value is in a sequence
  map_merge   # merges maps together
  map_replace   # performs key/value replacements on existing mapping
  repeat    # allows for dynamically transforming lists by iterating over the contents of one or more source lists and replacing list elements in the template
  resource_facade   # retrieves data in a parent provider template. A facade is a custom definition of a resource from a provider template
  str_replace   # constructs strings by providing a template string with placeholders and a list of mappings to assign values to those placeholders at runtime
  str_replace_strict    # similar to str_replace, only an error is raised if any params are not present in template
  str_replace_vstrict   # similar to str_replace, only an error is raised if any params are empty
  str_split   # allows for a string to be split into a list by providing an arbitrary delimiter
  yaql    # evaluates yaql expression on given data
  if      # returns corresponding value based on evaluation of a condition

For more details about these intrinsic functions, please see the following documentation: https://docs.openstack.org/heat/train/template_guide/hot_spec.html#get-attr



The list of supported condition functions is:

.. code-block:: text

  equals      # compares whether two values are equal
  get_param   # references input parameter of a template
  not         # acts as a NOT operator
  and         # acts as an AND operator
  or          # acts as an OR operator
  yaql        # evaluates yaql expression on given data
  contains    # checks whether a specific value is in a sequence


Pseudo Parameters
^^^^^^^^^^^^^^^^^^

As well as parameters defined by the template author. Heat creates three parameters for every stack:

.. code-block:: bash

  OS::stack_name  # stack name
  OS::stack_id    # stack identifier
  OS::project_id  # project identifier


These parameters are accessible using ``get_param`` function.


References
-----------

https://www.cisco.com/c/dam/en/us/products/collateral/cloud-systems-management/metacloud/newbie-tutorial-heat.pdf

https://docs.aws.amazon.com/AWSCloudFormation/latest/APIReference/Welcome.html?r=7078

https://docs.openstack.org/heat/train/developing_guides/architecture.html

https://docs.openstack.org/heat/train/template_guide/openstack.html

https://docs.openstack.org//heat/latest/doc-heat.pdf

https://docs.openstack.org/heat/train/template_guide/hot_spec.html#hot-spec

https://docs.ansible.com/ansible/latest/reference_appendices/YAMLSyntax.html
