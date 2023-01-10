===================
Create a Heat stack
===================

Heat is an orchestration component of OpenStack which allows templates to be used to define resources and to specify the relationships between resources in a stack. This page explains the basics of how to create a heat stack. For more detailed docs and examples, see the **Heat Advanced Material** pages at https://stfc-cloud-docs.readthedocs.io/en/latest/Heat/index.html.

To check Heat is available we can run the command:

.. code-block:: bash

  openstack stack list

This command returns the list of stacks in the user's project. If no stacks have been created, this command returns an empty line. If Heat is not installed, see the more detailed pages on Heat: https://stfc-cloud-docs.readthedocs.io/en/latest/Heat/IntroducingOpenStackHeat.html#heat-commands

To create a stack, we first need to write a template which will be used to launch the stack.

##########
Templates
##########

Templates are used to define the resources which should be created as part of a stack and the relationships between resources.
With the integration of other OpenStack components, Heat Templates allow the create of most OpenStack resource types.
This includes infrastructure resources such as instances, databases, security groups, users, etc.
The Heat Orchestration Template (HOT) is native to OpenStack for creating stacks. These templates are written in YAML.


Heat Orchestration Templates
############################

The structure of a Heat Orchestration Template is given as follows:

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

heat_template_version
#####################

This tells heat which format and features will be supported when creating the stack.

For example:

.. code-block:: yaml

  heat_template_version: 2018-03-02

Indicates that the Heat Template contains features which have been added and/or removed up to the Queens release of OpenStack.

The template version:

.. code-block:: yaml

  heat_template_version: 2018-08-31

Indicates that the Heat Template contains features which have been added and/or removed up to the Rocky release of OpenStack.
For templates being used in the current release of OpenStack used, this template version is used.

A list of template versions can be found here: https://docs.openstack.org/heat/train/template_guide/hot_spec.html#heat-template-version

parameter_groups
################

Parameter groups indicate how the input parameters are grouped. The order of the parameters are given in lists.
This section is not strictly compulsory for launching stacks, however it is useful for more complex templates where there are several input parameters to be used.

The syntax for parameter_groups is:

.. code-block:: yaml

  parameter_groups:
  - label: # label defining the group of parameters
    description: #description of parameter group
    parameters:
      - <parameter-1> #name of first parameter
      - <parameter-2> #name of second parameter
      - <parameter-3> #name of third parameter

Parameters
##########

The parameters section specifies the input parameters that have to be provided when the template is initalised. The syntax for parameters is of the form:

.. code-block:: yaml

  parameters:
    <param name>:
      type: <string | number | json | comma_delimited_list | boolean>
      label: <human-readable name of the parameter> #optional
      description: <description of the parameter> #optional
      default: <default value for parameter> #optional - this is used if the user does not specify a value.
      hidden: <true | false> #default option is false - this determines whether the parameter is hidden from the user if the user requests information about the stack.
      constraints: <parameter constraints> #optional - list of constraints to apply to the parameter. The stack will fail if the parameter values doe not comply to the constrains.
      immutable: <true | false> #default is false - this determines whether a parameter is updateable after the stack is running.
      tags: <list of parameter categories> #optional input - list of strings to specify the category of the parameter.

Resources
#########

This is a compulsory section and must contain at least one resource. This could be an instance, floating IP, Network, key pair, etc.

A list of the different OpenStack resources which can be used in a Heat template can be found here: https://docs.openstack.org/heat/latest/template_guide/openstack.html

.. code-block:: yaml

  resources:
    <resource ID>: #must be unique within the resources section of the template.
      type: <resource type> #e.g OS::Nova::Server, OS::Nova::Port, OS::Neutron::FloatingIPAssociation, etc.
      properties: #list of resource-specific properties that can be provided in place or via a function.
        <property name>: <property value>
        metadata: #optional
        <resource specific metadata>
        depends_on: <resource ID or list of ID> #optional
        update_policy: <update policy> #optional - this is given in the form of a nested dictionary.
        deletion_policy: <deletion policy> #optional - allowed deletion policies are Delete, Retain, and Snapshot
        external_id: <external resource ID> #optional - can define a resource which is external to the stack
        condition: <condition name or expression or boolean> #optional input - decides whether the resource should be created based on a given condition

Below is an example of the resource section for an instance.

.. code-block:: yaml

  my_instance: #name of the instance
    type: OS::Nova::Server
    properties:
      image: image_id #retrieves the image ID from image_id parameter
      flavor: flavor_id #retrieves the flavor ID from flavor_id parameter
      key_name: key_name #retrieves the key pair from key_name parameter
      networks:
        - network: network_name #define the internal network as Internal
      security_groups:
        - security_group_id


Outputs
#######

Outputs define the parameters that should be available to the user after a stack has been created.
This would be, for example, parameters such as the IP addresses of deployed instances, or the URL of web applications deployed as a stack. Each output is defined as a separate block within the outputs section:

.. code-block:: yaml

  outputs:
    <parameter name>:
      description: <description>
      value: <parameter value>
      condition: <condition name or expression or boolean>

Conditions
##########

The conditions section in the heat template defines at least one condition that is evaluated based on the input parameter values when a user creates or updates a stack. The conditions can be associated with resources, the properties of the resources and the output.

The syntax for conditions in the heat template is given by:

.. code-block:: yaml

  conditions:
    <condition_name_1>: {expression_1}
    <condition_name_2>: {expression_2}


#################
Example Template
#################

The following template (example-template.yaml) is for a stack containing a single instance.

.. code-block:: yaml

  heat_template_version: 2018-08-31 #OpenStack Rocky Version

  description: An example template which launches instances.

  parameter_groups: # Optional - helps to group parameters together
    - label: Instance parameters #human-readable label defining the associated group of parameters
      description: The parameters which are required to launch an instance. #description of parameter group
      parameters: #Parameters are given same order as launching an instance using openstack server create command
        - key_name #name of keypair to SSH into instance
        - image_id #name can be used as well, but it's better practice to use ID
        - flavor_id #name or ID, though it is better practice to use ID
        - security_group_id #security group for the instance (use the security group ID)
        #network will be defined inside resources

  parameters: #declares the parameters
    key_name:
      type: string
      default: <key-name>
      description: Key pair to use to be able to SSH into instance
    image_id:
      type: string
      default: <image-id> #Image ID
      description: The image for the instance will be IMAGE-NAME
    flavor_id:
      type: string
      default: <flavor-id> #Flavor ID
      description: The flavor for the instance will be FLAVOR-NAME
    security_group_id:
      type: string
      default: <security-group-id> #ID of the security group
      description: SECURITY-GROUP-NAME #this could be a default security group for example

  resources: #declares the template resources
    test_instance: #name of the instance
      type: OS::Nova::Server
      properties:
        image: { get_param: image_id } #retrieves the image ID from image_id parameter
        flavor: { get_param: flavor_id } #retrieves the flavor ID from flavor_id parameter
        key_name: { get_param: key_name } #retrieves the key pair from key_name parameter
        networks:
          - network: Internal #define the internal network as Internal
        security_groups:
          - { get_param: security_group_id }

Using a template similar to this one, we can launch a stack.

###############
Create a Stack
###############

Stacks can be launched using the OpenStack CLI. The syntax for creating a stack is:

.. code-block:: bash

  openstack stack create [-h] [-f {json,shell,table,value,yaml}]
                              [-c COLUMN] [—noindent] [—prefix PREFIX]
                              [—max-width <integer>] [—fit-width]
                              [—print-empty] [-e <environment>]
                              [-s <files-container>] [—timeout <timeout>]
                              [—pre-create <resource>] [—enable-rollback]
                              [—parameter <key=value>]
                              [—parameter-file <key=file>] [—wait]
                              [—poll SECONDS] [—tags <tag1,tag2…>]
                              [—dry-run] -t <template>
                              <stack-name>

For example, to create a stack using the template *example-template.yaml*:

.. code-block:: bash

  openstack stack create -t example-template.yaml example-stack

This should return something similar to the following:

.. code-block:: bash

  +---------------------+--------------------------------------------------+
  | Field               | Value                                            |
  +---------------------+--------------------------------------------------+
  | id                  | deda567a-4240-466d-9ac6-4bed4b848666             |
  | stack_name          | example-stack                                    |
  | description         | An example template which launches instances.    |
  | creation_time       | 2020-07-20T08:22:14Z                             |
  | updated_time        | None                                             |
  | stack_status        | CREATE_IN_PROGRESS                               |
  | stack_status_reason | Stack CREATE started                             |
  +---------------------+--------------------------------------------------+

Then the status of the stack can be checked using the command:

.. code-block:: bash

  openstack stack show <stack-id>


###############
Delete a Stack
###############

To delete a stack, use the command:

.. code-block:: bash

  openstack stack delete <stack-id>

**Note:** Any resources such as instances which have been created specifically for the stack will also be deleted.

################
Further Reading
################

For more detailed information on Heat, and to see some example stacks, refer to the advance Heat docs at https://stfc-cloud-docs.readthedocs.io/en/latest/Heat/index.html

###########
References
###########

https://docs.openstack.org/heat/train/template_guide/hot_guide.html

https://docs.openstack.org/heat/train/template_guide/hot_spec.html#hot-spec

https://www.cisco.com/c/dam/en/us/products/collateral/cloud-systems-management/metacloud/newbie-tutorial-heat.pdf
