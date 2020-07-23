===================
Create a Heat stack
===================

This document is in progress.

######
Heat
######
To check Heat is available we can run the command:

.. code-block:: bash

  openstack stack list

This command returns the list of stacks in the user's project. If no stacks have been created, this command returns an empty line.

To create a stack, we first need to write a template which will be used to launch the stack.

##########
Templates
##########
Templates are used to define the resources which should be created as part of a stack and the relationships between resources.
With the integration of other OpenStack components, Heat Templates allow the create of most OpenStack resource types.
This includes infrastructure resources such as instances, databases, security groups, users, etc.
The Heat Orchestration Template (HOT) is native to OpenStack for creating stacks. These templates are written in YAML.

#############################
Heat Orchestration Templates
#############################
The structure of a Heat Orchestration Template is given as follows:

.. code-block:: yaml

  heat_template_version: 2018-03-02 #OpenStack Version we want to use.
                                  #Here it is the Queens version

  description: #description of the template. What does the template do?

  parameter_groups: #declares the parameter group and order.
    #This is not a compulsory section, however it is useful for grouping
    #parameters together when building more complex templates.

  parameters: #declares the parameters for resources

  resources: #declares the template resources
  # e.g. instances, floating IPs, etc.

  outputs: #declares the output of the stack

  conditions: #declares any conditions on the stack

heat_template_version
#####################

This tells heat which format and features will be supported when creating the stack.
For example:

.. code-block:: yaml

  heat_template_version: 2018-03-02

Indicates that the Heat Template contains features which have been added an/or removed up to the Queens release of OpenStack.
A list of template versions can be found here: https://docs.openstack.org/heat/rocky/template_guide/hot_spec.html#hot-spec-intrinsic-functions

parameter_groups
################

Parameter groups allows users to specify how the input parameters should be grouped in the order that the parameters are provided in. This section is not strictly compulsory for launching stacks, however it is useful for more complex templates where there are several input parameters to be used.

The syntax for parameter_groups is:

.. code-block:: yaml

  parameter_groups:
    label: #human-readable label defining the associated group of parameters
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
      label: <human-readable name of the parameter> #optional input
      description: <description of the parameter> #optional input
      default: <default value for parameter> #optional - this is used if the user does not specify a value.
      hidden: <true | false> #Default option is false. This determines whether the parameter is hidden from the user if the user requests information about the stack.
      constraints: <parameter constraints> #Optional Input. List of constrains to apply to the parameter. The stack will fail if the parameter values doe not comply to the constrains.
      immutable: <true | false> #Default is false. This determines whether a parameter is updateable.
      tags: <list of parameter categories> #Optional input. list of strings to specify the catagory of the parameter.

Resources
#########

This is a compulsory section and must contain at least one resource. This could be an instance, floating IP, Network, key pair, etc.

.. code-block:: yaml

  resources:
    <resource ID>: #must be unique within the resources section of the template.
      type: <resource type> #e.g OS::Nova::Server (instance), OS::Nova::Port
      properties: #list of resource-specific properties that can be provided in place or via a function.
        <property name>: <property value>
        metadata: #optional input
        <resource specific metadata>
        depends_on: <resource ID or list of ID> #optional input
        update_policy: <update policy> #optional input
        deletion_policy: <deletion policy> #optional input. Default policy is to delete the physical resource when deleting a resource from the stack.
        external_id: <external resource ID> # cannot depend on other resources.
        condition: <condition name or expression or boolean> #optional input. Decides whether the resource should be created.

Below is an example of a resource being a single instance.

.. code-block:: yaml

  my_instance: #name of the instance
    type: OS::Nova::Server
    properties:
      image: image_id #retrieves the image ID from image_id parameter
      flavor: flavor_id #retrieves the flavor ID from flavor_id parameter
      key_name: key_name #retrieves the key pair from KeyName parameter
      networks:
        - network: network_name #define the internal network as Internal
      security_groups:
        - security_group_id


Outputs
#######

Outputs define the parameters that should be available to the user after a stack has been created. This would be, for example, parameters such as the IP addresses of deployed instances, or the URL of web applications deployed as a stack. Each output is defined as a separate block within the outputs section:

.. code-block:: yaml

  outputs:
    <parameter name>:
      description: <description>
      value: <parameter value>
      condition: <condition name or expression or boolean>

Conditions
##########

The condition section in the heat template defines at least one condition that is evaluated based on the input parameter values when a user creates or updates a stack. The conditions can be associated with resources, the properties of the resources and the output.

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

  heat_template_version: 2018-03-02 #OpenStack Queens Version

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
      label: <image-name>
      default: <image-id> #Image ID
      description: The image for the instance will be IMAGE-NAME
    flavor_id:
      type: string
      label: <flavor-name>
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

Stacks can be launched using the openstack CLI. The syntax for creating a stack is:

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

  openstack stack create -t example-template.yaml

This should return something similar to the following:

.. code-block:: bash

  +---------------------+--------------------------------------------------+
  | Field               | Value                                            |
  +---------------------+--------------------------------------------------+
  | id                  | deda567a-4240-466d-9ac6-4bed4b848666             |
  | stack_name          | example-stack                                    |
  | description         | A template to test launching a stack with one VM |
  | creation_time       | 2020-07-20T08:22:14Z                             |
  | updated_time        | None                                             |
  | stack_status        | CREATE_IN_PROGRESS                               |
  | stack_status_reason | Stack CREATE started                             |
  +---------------------+--------------------------------------------------+

Then the status of the stack can be checked using the command:

.. code-block:: bash

  openstack stack show <stack-id>

You should get the details of the stack similar to the following:


###############
Delete a Stack
###############

To delete a stack, use the command:

.. code-block:: bash

  openstack stack delete <stack-id>

**Note:** Any resources such as instances which have been created specifically for the stack will also be deleted.




###########
References
###########
https://docs.openstack.org/heat/queens/template_guide/hot_guide.html

https://www.cisco.com/c/dam/en/us/products/collateral/cloud-systems-management/metacloud/newbie-tutorial-heat.pdf
