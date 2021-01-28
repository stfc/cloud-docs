============================
AutoScaling in a Heat Stack
=============================

Sometimes we may need to have a stack that can respond when a group of servers are using a lot or little resources, such as memory usage. For example if a group of servers exceed a given memory usage threshold, we want that group of resources to scale up.
This documentation will go through autoscaling, and how autoscaling could be implemented in a Heat stack.

To create an autoscaling stack we need:

- **AutoScaling Group:** A group of servers defined so that the number of servers in the group and be increased or decreased.

- **Alarms:** Alarms created using OpenStack Aodh to monitor the resource usage of the VMs in the autoscaling group. For example, we could create an alarm to monitor memory usage and alarm if the autoscaling group exceeds the alarm's threshold.

- **Scaling Policies:** Policies which are executed when an Aodh Alarm is triggered. When an alarm is triggered, the scaling policy attached to that alarm will instruct the autoscaling group to change in size, either increasing or decreasing the number of VMs.


Heat Resources
---------------

This section will cover the resources available in Heat that are required for creating an autoscaling stack.

AutoScaling involves resources from:

- **Heat:** For creating an autoscaling group and defining the scaling policies
- **Aodh:** For alarm creation
- **Gnocchi:** For metrics that are used in threshold alarms


OS::Heat::AutoScalingGroup
~~~~~~~~~~~~~~~~~~~~~~~~~~~

This is an autoscaling group which can scale resources. This group can create the desired number of similar resources and we can define the minimum and maximum count for the given resource.

.. code-block:: yaml

  the_resource:
    type: OS::Heat::AutoScalingGroup
    properties:
      #required
      max_size: Integer # maximum number of resources in the group
      min_size: Integer # minimum number of resources in the group
      resource: {...} # resource definition for the resources in the group, written in HOT (Heat Orchestrated Template) format
      #optional
      desired_capacity: Integer # desired initial number of resources
      cooldown: Integer # cool down period in seconds
      rolling_updates: {"min_in_service": Integer, "max_batch_size": Integer, "pause_time": Number} # policy for rolling updates in the group, defaults to: {"min_in_service": 0, "max_batch_size": 1, "pause_time": 0}
        # min_in_service: minimum number of resources in service while rolling updates are executed
        # max_batch_size: maximum number of resources to replace at once
        # pause_time: number of seconds to wait between batches of updates


For example:

.. code-block:: yaml

  autoscaling-group:
    type: OS::Heat::AutoScalingGroup
    properties:
      min_size: 1
      max_size: 3
      resource:
        type: server.yaml #Refers to a Heat Template for creating a VM
        properties:
          flavor: {get_param: flavor}
          image: {get_param: image}
          key_name: {get_param: key_name}
          network: {get_param: network}
          metadata: {"metering.server_group": {get_param: "OS::stack_id"}}


OS::Heat::ScalingPolicy
~~~~~~~~~~~~~~~~~~~~~~~~


.. code-block:: yaml

  the_resource:
    type: OS::Heat::ScalingPolicy
    properties:
      # required
      adjustment_type: String # Type of adjustment. Allowed values: “change_in_capacity”, “exact_capacity”, “percent_change_in_capacity”
      auto_scaling_group_id: String # AutoScaling Group ID to apply policy to
      scaling_adjustment: Number # Size of adjustment
      # Optional
      cooldown: Number # cooldown period, in seconds
      min_adjustment_step: Integer # minimum number of resources that are added or removed when the AutoScalingGroup scales up or down. Only used if specifying percent_change_in_capacity for adjustment_type property

For example:

.. code-block:: yaml

  scaleup_policy:
    type: OS::Heat::ScalingPolicy
    properties:
      adjustment_type: change_in_capacity
      auto_scaling_group_id: {get_resource: autoscaling-group}
      cooldown: 60
      scaling_adjustment: 1

  scaledown_policy:
    type: OS::Heat::ScalingPolicy
    properties:
      adjustment_type: change_in_capacity
      auto_scaling_group_id: {get_resource: autoscaling-group}
      cooldown: 60
      scaling_adjustment: -1



OS::Aodh::GnocchiAggregationByResourcesAlarm
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

This resource creates an alarm as an aggregation of resources alarm.
This alarm is a threshold alarm monitoring the aggregated metrics of the members of the autoscaling group defined above. Gnocchi provides the metrics which Aodh uses to determine whether an alarm should be triggered.

.. code-block:: yaml

    the_resource:
      type: OS::Aodh::GnocchiAggregationByResourcesAlarm
      properties:
        # required
        metric: String # metric name watched by the alarm
        query: String # query to filter the metrics
        resource_type: String # resource type
        threshold: Number # threshold to evaluate against

        # optional
        aggregation_method: String # method to compare to the threshold
        alarm_actions: [Value, Value, ...] # list of webhooks to invoke when state transitions to alarm
        alarm_queues: [String, String, ...] # list of Zaqar queues to post to when state transitions to alarm
        comparison_operator: String # operator used to compare specified statistic with threshold. Allowed values: “le”, “ge”, “eq”, “lt”, “gt”, “ne”
        description: String # alarm description
        enabled: Boolean # Defaults to true. Determines if alarm evaluation is enabled
        evaluation_periods: Integer # number of periods to evaluate over
        granularity: Integer # time range in seconds
        insufficient_data_actions: [Value, Value, ...] # list of webhooks to invoke when state transitions to insufficient data
        insufficient_data_queues: [String, String, ...] # list of Zaqar queues to post to when state transitions to alarm
        ok_actions: [Value, Value, ...] # list of webhooks to invoke when state transitions to ok
        ok_queues: [String, String, ...] # list of Zaqar queues to post to when state transitions to ok
        repeat_actions: Boolean # Defaults to True. False to trigger actions when the threshold is reached AND the alarm has changed state
        severity: String # severity of alarm. Allowed values: “low”, “moderate”, “critical”
        time_constraints: [{"name": String, "start": String, "description": String, "duration": Integer, "timezone": String}, {"name": String, "start": String, "description": String, "duration": Integer, "timezone": String}, ...] # Describe time constraints for alarm, defaults to []
          # description: description for time constraints
          # duration: duration for time constraint
          # name: name for time constraint
          # start: start time for time constraint. A CRON expression property
          # timezone: Timezone for the time constraint.

For example, for our autoscaling stack we could define the alarms in the following way:

.. code-block:: yaml

    memory_alarm_high:
      type: OS::Aodh::GnocchiAggregationByResourcesAlarm
      properties:
        description: Scale up if memory > 1000 MB
        metric: memory.usage
        aggregation_method: mean
        granularity: 300
        evaluation_periods: 1
        threshold: 1000
        resource_type: instance
        comparison_operator: gt
        query:
          list_join:
            - ''
            - - {'=': {server_group: {get_param: "OS::stack_id"}}}
        alarm_actions:
          - get_attr: [scaleup_policy, signal_url]

    memory_alarm_low:
      type: OS::Aodh::GnocchiAggregationByResourcesAlarm
      properties:
        description: Scale down if memory < 200MB
        metric: memory.usage
        aggregation_method: mean
        granularity: 300
        evaluation_periods: 1
        threshold: 200
        resource_type: instance
        comparison_operator: lt
        query:
          list_join:
            - ''
            - - {'=': {server_group : {get_param: "OS::stack_id"}}}
        alarm_actions:
          - get_attr: [scaledown_policy, signal_url]




References
------------

https://docs.openstack.org/heat/train/template_guide/openstack.html

https://ibm-blue-box-help.github.io/help-documentation/heat/autoscaling-with-heat/

https://github.com/openstack/heat-templates/blob/master/hot/autoscaling.yaml

https://bhujaykbhatta.wordpress.com/2018/01/18/auto-scaling-in-openstack-using-heat-gnocchi-and-aodh/
