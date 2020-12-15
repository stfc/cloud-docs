==================
Create Aodh Alarms
==================

This document is in progress.

Aodh alarms can be created using the Openstack CLI or by using Aodh Resources in a heat template.

Overview
########

Alarms provide monitoring-as-a-service for user's resources running on Openstack.
One application of these alarms is to create autoscaling stacks, where alarms determine whether a scale-up or scale-down policy is applied to a group of instances.

Gnocchi metrics can be used to create threshold alarms, for example.

The Aodh and gnocchi CLIs can be installed using:

.. code-block:: bash

  pip install aodhclient
  pip install gnocchiclient

################
Types of alarms
################

There are three different kinds of alarms:

- Threshold: Alarms are triggered when a threshold have been met.

  - Valid Threshold Alarms:
  
    - gnocchi_resources_threshold
    
    - gnocchi_aggregation_by_metrics_threshold
    
    - gnocchi_aggregation_by_resources_threshold


- Composite Alarms: Alarms which have been defined with multiple triggering conditions.


- Event Alarms: Alarms when an event has occurred, such as an instance being powered down or in error state.

Alarms can have three different states:

- **ok:** The alarm rule has been defined as False
- **alarm:** The alarm rule has been defined as True
- **insufficient data:** There are not enough data points in order to determine the state of the alarm.



Alarm Commands
###############

.. code-block:: bash

  openstack alarm list #lists user-created alarms

  openstack alarm create <options> # create an alarm

  openstack alarm update <options> <alarm-id> #update the properties of an alarm

  openstack delete <alarm-id> #delete an alarm

  openstack alarm show <alarm-id> #show details of an alarm

  openstack alarm-history show <alarm-id> #see history of the alarm

  openstack alarm-history search #search history for all alarms based on query

  openstack alarm state set #set the state of an alarm

  openstack alarm state get #get state of an alarm

To list user-created alarms, use the command:

.. code-block:: bash

  openstack alarm list


This returns an empty line if no alarms have been created, or returns a list of alarms such as:

.. code-block:: bash

  +--------------------------------------+-----------------------------+--------------------+-------------------+----------+---------+
  | alarm_id                             | type                        | name               | state             | severity | enabled |
  +--------------------------------------+-----------------------------+--------------------+-------------------+----------+---------+
  | alarm-id-1                           | event                       | instance_off       | alarm             | low      | True    |
  | alarm-id-2                           | gnocchi_resources_threshold | memory_use_test    | ok                | low      | True    |
  +--------------------------------------+-----------------------------+--------------------+-------------------+----------+---------+


################
Create an Alarm
################

To create an alarm, use the command:

.. code-block:: bash

  openstack alarm create <options>

Create Alarm Options
####################

.. code-block:: bash

  openstack alarm create [-h] [-f {json,shell,table,value,yaml}]
                              [-c COLUMN] [--noindent] [--prefix PREFIX]
                              [--max-width <integer>] [--fit-width]
                              [--print-empty] --name <NAME> -t <TYPE>
                              [--project-id <PROJECT_ID>]
                              [--user-id <USER_ID>]
                              [--description <DESCRIPTION>] [--state <STATE>]
                              [--severity <SEVERITY>] [--enabled {True|False}]
                              [--alarm-action <Webhook URL>]
                              [--ok-action <Webhook URL>]
                              [--insufficient-data-action <Webhook URL>]
                              [--time-constraint <Time Constraint>]
                              [--repeat-actions {True|False}]
                              [--query <QUERY>]
                              [--comparison-operator <OPERATOR>]
                              [--evaluation-periods <EVAL_PERIODS>]
                              [--threshold <THRESHOLD>]
                              [--event-type <EVENT_TYPE>] [-m <METER NAME>]
                              [--period <PERIOD>] [--statistic <STATISTIC>]
                              [--granularity <GRANULARITY>]
                              [--aggregation-method <AGGR_METHOD>]
                              [--metric <METRIC>]
                              [--resource-type <RESOURCE_TYPE>]
                              [--resource-id <RESOURCE_ID>]
                              [--composite-rule <COMPOSITE_RULE>]
                              [--stack-id <STACK_NAME_OR_ID>]
                              [--pool-id <LOADBALANCER_POOL_NAME_OR_ID>]
                              [--autoscaling-group-id <AUTOSCALING_GROUP_NAME_OR_ID>]


**--name:** Alarm name - this should be unique to the alarm in the project.

**--type:** Type of alarm - event, composite, threshold, gnocchi_resources_threshold, gnocchi_aggregation_by_metrics_threshold, gnocchi_aggregation_by_resources_threshold, loadbalancer_member_health.

**--description:** Free text description of the alarm.

**--state:** State of the alarm - ok, alarm, insufficient data.

**--severity:** Severity of the alarm - low, moderate, critical.

**--enabled:** Determine whether the alarm is evaluated. True if alarm evaluation is enabled.



**Actions when alarm changes state:**



**--alarm-action:** [Webhook URL] URL to invoke when alarm transitions to alarm state.

**--ok-action:** [Webhook URL] URL to invoke when alarm transitions to ok state.

**--insufficient-data-action:** [Webhook URL] URL to invoke when alarm transitions to insufficient data state.

These actions may be used multiple times.

**--time-constraint:**  Only evaluate the alarm if it is within the time constraint. Start point(s) specified with a cron expression, and duration in seconds. Can be specified multiple times for multiple constraints. Format:  :bash:`name=<CONSTRAINT_NAME>;start=<CRON>;duration=<SECONDS>;[description=<DESCRIPTION>;[timezone=<IANA Timezone>]]`

**--repeat-actions:** [Default to False] Determines whether actions should be repeatedly notified why the alarm remains in target state.


**Alarm Rules**


**--query:**
  - Threshold or Event Type:  key[op]data_type::value; list. data_type is optional, but if supplied must be string, integer, float, or boolean
  - gnocchi_aggregation_by_resources_threshold: need to specify a complex query json string, like: :bash:`{"and": [{"=": {"ended_at": null}}, ...]}`

**--comparison-operator:** Operator to compare with - lt, le, eq, ne, ge, gt.

**--evaluation_periods:** Number of periods to evaluate over.

**--threshold:** Threshold to evaluate against.


**Event Alarm**


**--event-type:** Event type to evaluate against.


**Threshold Alarm**


**-m, --meter-name:** Meter to evaluate against.

**--period:** length of each period (seconds) to evaluate over.

**--statistic:** Statistic to evaluate - max, min, avg, sum, count.


**Common gnocchi alarm rules**


**--aggregation-method:** The aggregation_method to compare to the threshold.

**--metric, --metrics:** The metric id or name depending of the alarm type


**Gnocchi resource threshold alarm**:


**--resource-type:** The type of resource.

**--resource-id:** The id of a resource.


**Composite Alarm:**


**--composite-rule:** Composite threshold rule with JSON format, the form can be a nested dict which combine threshold/gnocchi rules by "and", "or". For example, the form is like: {"or":[RULE1, RULE2, {"and": [RULE3, RULE4]}]}, The RULEx can be basic threshold rules but must include a "type" field, like this: {"threshold": 0.8,"meter_name":"cpu_util","type":"threshold"}


**Loadbalancer member health alarm**



**--stack-id:** Name or ID of the root / top level Heat stack containing the loadbalancer pool and members. An update will be triggered on the root Stack if an unhealthy member is detected in the loadbalancer pool.

**--pool-id:** Name or ID of the loadbalancer pool for which the health of each member will be evaluated.

**--autoscaling-group-id:** ID of the Heat autoscaling group that contains the loadbalancer members. Unhealthy members will be marked as such before an update is triggered on the root stack.



After the alarm is created, you should get the alarm details similar to this one:

.. code-block:: bash

  +---------------------------+------------------------------------------------------------------+
  | Field                     | Value                                                            |
  +---------------------------+------------------------------------------------------------------+
  | alarm_actions             | []                                                               |
  | alarm_id                  | ALARM_ID                                                         |
  | description               | Instance powered OFF                                             |
  | enabled                   | True                                                             |
  | event_type                | compute.instance.power_off.*                                     |
  | insufficient_data_actions | []                                                               |
  | name                      | power_off_alarm                                                  |
  | ok_actions                | []                                                               |
  | project_id                | PROJECT_ID                                                       |
  | query                     | traits.instance_id = INSTANCE_ID                                 |
  | repeat_actions            | False                                                            |
  | severity                  | low                                                              |
  | state                     | insufficient data                                                |
  | state_reason              | Not evaluated yet                                                |
  | state_timestamp           | YYYY-MM-DDTHH:MM:SS                                              |
  | time_constraints          | []                                                               |
  | timestamp                 | YYYY-MM-DDTHH:MM:SS                                              |
  | type                      | event                                                            |
  | user_id                   | USER_ID                                                          |
  +---------------------------+------------------------------------------------------------------+





Alarms: Threshold, Event, Composite
###################################


Threshold Alarm
###############

Threshold alarms will change to alarm state when a threshold has been met.

The threshold is based of the value which is returned by the metric.

The metrics for instances are:

+----------------------------+-------------------+
| Metric                     | Unit              |
+============================+===================+
| network.incoming.bytes     | B                 |
+----------------------------+-------------------+
| network.incoming.packets   | packet            |
+----------------------------+-------------------+
| network.outgoing.bytes     | B                 |
+----------------------------+-------------------+
| network.outgoing.packets   | packet            |
+----------------------------+-------------------+
| disk.device.read.bytes     | B                 |
+----------------------------+-------------------+
| disk.device.read.requests  | request           |
+----------------------------+-------------------+
| disk.device.write.bytes    | B                 |
+----------------------------+-------------------+
| disk.device.write.requests | request           |
+----------------------------+-------------------+
| cpu                        | ns                |
+----------------------------+-------------------+
| disk.ephemeral.size        | GB                |
+----------------------------+-------------------+
| disk.root.size             | GB                |
+----------------------------+-------------------+
| memory.usage               | MB                |
+----------------------------+-------------------+
| memory                     | MB                |
+----------------------------+-------------------+
| vcpus                      | vcpu              |
+----------------------------+-------------------+

These metrics can be used when defining a gnocchi threshold alarm.

    **Note:** The alarm granularity must match the granularities of the metric configured in Gnocchi, otherwise the alarm will only return an 'insufficient data' state.


**Granularity:** This refers to the time interval (in seconds) in which data is collected.


The following example shows how a threshold alarm can be created for an instance.
This alarm is triggered when the memory usage exceeds 10%. However,
as the memory usage meter only measures the number of MBs of memory being used.

In order to attach a 10% memory usage alarm to an instance which has 1GB RAM, the threshold is simply
102 MB (which is approximately 10% of 1024MB).


.. code-block:: bash

  openstack alarm create --name memory_use_test
  --type gnocchi_resources_threshold
  --description "A test alarm which alarms when the memory usage exceeds 10% of RAM. For this VM, this would be 102MB."
  --metric memory.usage
  --threshold 102
  --comparison-operator gt
  --aggregation-method mean
  --granularity 300
  --evaluation-periods 2
  --resource-id <resource-id>
  --resource-type instance

So if, in two five minute evaluation periods the average memory usage is greater than 102MB, the alarm state will transition to alarm.
If the memory usage is below 102MB, the alarm state remains in the ok state.

**Q:** Can CPU utilization alarms be created?

**A:** Unfortunately, the *cpu_util* meter has been deprecated since the Stein release and so it is not possible to create alarms which monitor
the CPU utilization of instances.

**Q:** What about the cpu meter? Can the data from that meter be used and converted into % for CPU utilization?

**A:** Although the CPU is monitored it is measured in ns and the *openstack create alarm* command does not allow
operations to be performed on the meter in the --meter option. However, CPU utilization can be calculated manually
using the gnocchi command:

.. code-block:: bash

 gnocchi aggregates '(* (/ (aggregate rate:mean (metric cpu mean)) 60000000000) 100)' id=INSTANCE_ID


Event Alarm
############
The following example is an event alarm which transitions to 'alarm state' when an instance has no power (or has been powered off).

.. code-block:: bash

  openstack alarm create --type event \
  --name instance_off \
  --description 'Instance powered OFF' \
  --event-type "compute.instance.power_off.*" \ #event to monitor
  --enable True
  --query "traits.instance_id=string::INSTANCE_ID"

**Q:** The event alarm seems to be stuck in 'insufficient data' state and states that is has not been evaluated yet.

**A:** Unlike threshold alarms, event alarms will only change state when a specific event has occurred. This means
that event alarms will only transition to an alarm state. This also means that event alarms do not transition to ok state either.

**Q:** When an event has happened and the alarm has fired, will the alarm reset?

**A:** Event alarms are not reset automatically, so when they are in alarm state, they will stop beind g evaluated.
 For a power alarm on an instance, once the instance has been powered on the alarm state will need to be changed manually in order for the alarm to be evaluated again.
 When the state has been changed to 'insufficient data' or 'ok' using *opestack alarm update* or *openstack alarm state set*, the alarm will be monitored again and the alarm will move to 'alarm' state
 if the event occurs again.


Composite Alarms
################

Composite alarms use a combination of defined rules to determine the state of an alarm. These can be a combination of rules about the threshold of metrics being exceeded, or whether
a combination of events has occurred.

.. code-block:: bash

  openstack alarm create
  --name composite-alarm-test \
  --type composite \
  --composite-rule '{"or": [{"threshold": 500, "metric": "memory.usage", \
  "type": "gnocchi_resources_threshold", "resource_id": INSTANCE_ID1, \
  "resource_type": "instance", "aggregation_method": "last"}, \
  {"threshold": 500, "metric": "memory.usage", \
  "type": "gnocchi_resources_threshold", "resource_id": INSTANCE_ID2, \
  "resource_type": "instance", "aggregation_method": "last"}]}' \

This creates an alarm which will fire if either *INSTANCE_ID1* **or** *INSTANCE_ID2* uses more than 500MB memory.


References
##########

Aodh Alarms: https://docs.openstack.org/aodh/train/admin/telemetry-alarms.html

Gnocchi Documentation: https://gnocchi.xyz/stable_4.2/rest.html
