==============
Create a Heat stack
==============

Documentations coming soon

Work In Progress

######
Heat
######
To check Heat is available we can run the command:

.. .. code-block:: bash

  openstack stack list

This command returns the list of stacks in the user's project. If no stacks have been created, this command returns an empty line.

To create a stack, we first need to write a template which will be used to launch the stack.

######
Templates
######
Templates are used to define the resources which should be created as part of a stack and the relationships between resources.
With the integration of other OpenStack components, Heat Templates allow the create of most OpenStack resource types.
This includes infrastructure resources such as instances, databases, security groups, users, etc.
The Heat Orchestration Template (HOT) is native to OpenStack for creating stacks. These templates are written in YAML.

######
Heat Orchestration Templates
######
The structure of a Heat Orchestration Template is given as follows:

.. code-block:: YAML

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
######
This tells heat which format and features will be supported when creating the stack.
For example:
.. code-block:: YAML

  heat_template_version: 2018-03-02

Indicates that the Heat Template contains features which have been added an/or removed up to the Queens release of OpenStack.

######
References
#######
https://docs.openstack.org/heat/queens/template_guide/hot_guide.html
