Monitoring Virtual Machines: Gnocchi and OpenStack Aodh
=======================================================

> This document is a brief introduction to Aodh and Gnocchi.

We can monitor virtual machines by using alarms which fire when a threshold for a given measurement (e.g memory usage) has been met, when a VM has unexpectedly powered off, or when a VM has entered an error state.

In OpenStack, Aodh provides the ability to create, update and assign alarms to virtual machines.

To use alarms to monitor specific metrics from a virtual machine, Aodh uses metrics provided by Gnocchi.

Command Line Clients
^^^^^^^^^^^^^^^^^^^^

The command line clients for Aodh and Gnocchi can be installed using pip:

.. code-block:: bash

  pip install aodhclient
  pip install gnocchiclient


Aodh
----

Aodh is the telemetry alarming service which triggers alarm when the collected data or metrics have met or broken defined rules.

Alarms are governed by defined rules and have three states that they can enter:
  - **ok** - the rule has been evaluated as *false*
  - **alarm** - the rule has been evaluated as *true*
  - **insufficient_data** - there is not enough data available to meaningfully determine the alarm state.

There are three types of alarms which can be created using Aodh:
  - **Threshold Alarms:** these alarms move to an alarm state when a given threshold has been met or exceeded. An example of a threshold alarm would be a memory usage alarm, which uses the memory usage metric of the specific VM to determine the alarm's state. These alarms will move back to an **ok** state when the threshold is no longer being exceeded.

  - **Event Alarms:** these alarms move to an **alarm** state when a given event has occurred to a resource e.g a VM has unexpectedly powered off. These alarms only move to the **alarm** state once and will need their status updated manually to 'reset' the alarm.

    > **Note:** Event alarms will always start in an **insufficient_data** state once created and will only change state when the event has occurred.


  - **Composite Alarms:** these alarms use a combination of rules to determine the state of an alarm. These rules are combined with logical operators.

Please see the **_Create Aodh Alarms_** documentation for more information on how to create Aodh Alarms in the command line.

Gnocchi
-------

Gnocchi is a time series database which process data from the telemetry service Ceilometer. It correlates measurements from Ceilometer to **resources** and **metrics**.

**Metrics** are measurable quantities (e.g CPU time, memory usage etc) which are sampled from objects called **resources**. These resources could be instances (VMs), storage volume etc.

The gnocchi CLI enables us to use openstack commands to access and work with some of the metrics being stored.

.. code-block:: bash

  openstack metric <command>

  #The complete list of metric commands can be viewed using
  openstack metric --help


Metrics
^^^^^^^

A metric is a measurable resource property. This property could be memory usage, the amount of CPU time used, the number of vCPUs etc.

Metrics consist of:
  - **Name:** a unique name for the metric
  - **Unit:** e.g MB, ns, B/s
  - **Resource ID:** the resource where the measurement is from
  - **Unique User ID (UIID):** a unique identifier for the metric
  - **Archive Policy:** The policy for aggregating and storing the measurement for the metric.

> In OpenStack, Ceilometer automatically populates Gnocchi with metrics and resources.

The list of measurements from Ceilometer that Gnocchi can aggregate and store data from can be found here:

https://docs.openstack.org/ceilometer/train/admin/telemetry-measurements.html

> **Note:** Since the Stein Release of OpenStack, the CPU utilization meter has been deprecated and the meter cpu_util cannot be used. However, gnocchi does provide a way to get the CPU utilization of a resource manually.

For example, metrics which are collected from VMs are:

.. code-block:: bash

  +----------------------------+-------------------+
  | Metric                     | Unit              |
  +============================+===================+
  | network.incoming.bytes     | B                 |
  | network.incoming.packets   | packet            |
  | network.outgoing.bytes     | B                 |
  | network.outgoing.packets   | packet            |
  | disk.device.read.bytes     | B                 |
  | disk.device.read.requests  | request           |
  | disk.device.write.bytes    | B                 |
  | disk.device.write.requests | request           |
  | cpu                        | ns                |
  | disk.ephemeral.size        | GB                |
  | disk.root.size             | GB                |
  | memory.usage               | MB                |
  | memory                     | MB                |
  | vcpus                      | vcpu              |
  +----------------------------+-------------------+


CPU Utilization
'''''''''''''''''

Although the *cpu_util* meter has been deprecated since the OpenStack Stein release, we can use gnocchi to calculate the CPU utilization of a VM manually.

We can use the command `gnoochi agggregates <options>` to do this.

To calculate the CPU utilization of a VM, we can use the following command:

.. code-block:: bash

  gnocchi aggregates '(* (/ (aggregate rate:mean (metric cpu mean)) 60000000000) 100)' id=INSTANCE_ID


This will return a table similar to the following:

.. code-block:: bash

  +------------+---------------------------+-------------+--------------------+
  | name       | timestamp                 | granularity |              value |
  +------------+---------------------------+-------------+--------------------+
  | aggregated | 2020-07-13T07:45:00+00:00 |       300.0 |  3.266666666666666 |
  | aggregated | 2020-07-13T07:50:00+00:00 |       300.0 | 1.1666666666666667 |
  | aggregated | 2020-07-13T08:00:00+00:00 |       300.0 | 2.8666666666666667 |
  | aggregated | 2020-07-13T08:15:00+00:00 |       300.0 | 25.866666666666667 |
  | aggregated | 2020-07-13T08:25:00+00:00 |       300.0 |  2.216666666666667 |
  | aggregated | 2020-07-13T08:30:00+00:00 |       300.0 |               1.05 |
  | aggregated | 2020-07-13T08:35:00+00:00 |       300.0 | 1.0333333333333332 |
  | aggregated | 2020-07-13T08:40:00+00:00 |       300.0 |               1.05 |
  | aggregated | 2020-07-13T08:45:00+00:00 |       300.0 |               1.95 |
  | aggregated | 2020-07-13T08:50:00+00:00 |       300.0 | 1.1666666666666667 |
  | aggregated | 2020-07-13T08:55:00+00:00 |       300.0 | 1.1333333333333333 |
  | aggregated | 2020-07-13T09:00:00+00:00 |       300.0 | 1.7333333333333332 |
  | aggregated | 2020-07-13T09:05:00+00:00 |       300.0 | 1.1166666666666667 |
  | aggregated | 2020-07-13T09:10:00+00:00 |       300.0 | 1.2166666666666666 |
  | aggregated | 2020-07-13T09:15:00+00:00 |       300.0 | 22.916666666666664 |
  | aggregated | 2020-07-13T09:20:00+00:00 |       300.0 | 1.0999999999999999 |
  | aggregated | 2020-07-13T09:25:00+00:00 |       300.0 | 0.9833333333333333 |
  | aggregated | 2020-07-13T09:30:00+00:00 |       300.0 | 1.0666666666666667 |
  | aggregated | 2020-07-13T09:35:00+00:00 |       300.0 |                1.0 |
  | aggregated | 2020-07-13T09:40:00+00:00 |       300.0 | 1.0833333333333335 |
  | aggregated | 2020-07-13T09:45:00+00:00 |       300.0 | 1.8833333333333333 |
  | aggregated | 2020-07-13T09:50:00+00:00 |       300.0 |               1.25 |
  | aggregated | 2020-07-13T09:55:00+00:00 |       300.0 | 1.0999999999999999 |
  | aggregated | 2020-07-13T10:00:00+00:00 |       300.0 | 1.7833333333333332 |
  | aggregated | 2020-07-13T10:05:00+00:00 |       300.0 | 1.1833333333333333 |
  | aggregated | 2020-07-13T10:10:00+00:00 |       300.0 | 1.2333333333333334 |
  | aggregated | 2020-07-13T10:15:00+00:00 |       300.0 |              22.55 |
  | aggregated | 2020-07-13T10:20:00+00:00 |       300.0 | 1.0833333333333335 |
  | aggregated | 2020-07-13T10:25:00+00:00 |       300.0 | 1.0666666666666667 |
  | aggregated | 2020-07-13T10:30:00+00:00 |       300.0 | 1.1666666666666667 |
  | aggregated | 2020-07-13T10:35:00+00:00 |       300.0 |               1.15 |
  +------------+---------------------------+-------------+--------------------+


Archive Policies
^^^^^^^^^^^^^^^^

Archive policies are linked to every metric for each resource. These policies determines how many data points to collect over a given time period and the method for aggregating this data.

To view the list of archive policies, we can use the command:


.. code-block:: bash

  openstack metric archive-policy list


This will return a table listing each archive policy and how the policies are defined. The definitions given to each policy determines how raw datapoints for metrics from OpenStack Ceilometer are collected and aggregated.

.. code-bock:: bash

  +----------------------+-------------+-----------------------------------------------------------------------+---------------------------------+
  | name                 | back_window | definition                                                            | aggregation_methods             |
  +----------------------+-------------+-----------------------------------------------------------------------+---------------------------------+
  | bool                 |        3600 | - points: 31536000, timespan: 365 days, 0:00:00, granularity: 0:00:01 | last                            |
  | ceilometer-high      |           0 | - points: 3600, timespan: 1:00:00, granularity: 0:00:01               | mean                            |
  |                      |             | - points: 1440, timespan: 1 day, 0:00:00, granularity: 0:01:00        |                                 |
  |                      |             | - points: 8760, timespan: 365 days, 0:00:00, granularity: 1:00:00     |                                 |
  | ceilometer-high-rate |           0 | - points: 3600, timespan: 1:00:00, granularity: 0:00:01               | rate:mean, mean                 |
  |                      |             | - points: 1440, timespan: 1 day, 0:00:00, granularity: 0:01:00        |                                 |
  |                      |             | - points: 8760, timespan: 365 days, 0:00:00, granularity: 1:00:00     |                                 |
  | ceilometer-low       |           0 | - points: 8640, timespan: 30 days, 0:00:00, granularity: 0:05:00      | mean                            |
  | ceilometer-low-rate  |           0 | - points: 8640, timespan: 30 days, 0:00:00, granularity: 0:05:00      | rate:mean, mean                 |
  | high                 |           0 | - points: 3600, timespan: 1:00:00, granularity: 0:00:01               | std, count, min, max, sum, mean |
  |                      |             | - points: 10080, timespan: 7 days, 0:00:00, granularity: 0:01:00      |                                 |
  |                      |             | - points: 8760, timespan: 365 days, 0:00:00, granularity: 1:00:00     |                                 |
  | low                  |           0 | - points: 8640, timespan: 30 days, 0:00:00, granularity: 0:05:00      | std, count, min, max, sum, mean |
  | medium               |           0 | - points: 10080, timespan: 7 days, 0:00:00, granularity: 0:01:00      | std, count, min, max, sum, mean |
  |                      |             | - points: 8760, timespan: 365 days, 0:00:00, granularity: 1:00:00     |                                 |
  +----------------------+-------------+-----------------------------------------------------------------------+---------------------------------+



We can view details for an archive policy using the command:

.. code-block:: bash

  openstack metric archive-policy show <name>


This will return a table with information about the named archive policy. For example:

.. code-block:: bash

  openstack archive-policy show ceilometer-high


This table shows each archive-policy and how the raw datapoints for each metric is stored.
As an example, let's view the details for one of the archive policies.

.. code-block:: bash

  openstack archive-policy show  ceilometer-high

  # Output for this archive policy

  +---------------------+-------------------------------------------------------------------+
  | Field               | Value                                                             |
  +---------------------+-------------------------------------------------------------------+
  | aggregation_methods | mean                                                              |
  | back_window         | 0                                                                 |
  | definition          | - points: 3600, timespan: 1:00:00, granularity: 0:00:01           |
  |                     | - points: 1440, timespan: 1 day, 0:00:00, granularity: 0:01:00    |
  |                     | - points: 8760, timespan: 365 days, 0:00:00, granularity: 1:00:00 |
  | name                | ceilometer-high                                                   |
  +---------------------+-------------------------------------------------------------------+


In Gnocchi, **granularity** refers to the time interval between each aggregated data point. We can see from this table that for metrics collected using the archive policy ceilometer-high:

- The **mean** is stored for each interval.
- Stores **one hour** of data in **one second** intervals. (3600 data points)
- Stores **one day**  of data in **one minute** intervals. (1440 data points)
- Stores **one year** of data in **one hour** intervals. (8760 data points)

> **Note:** When creating threshold alarms which monitors metrics, it is important to check which *archive policy* they are using to collect the data. This is because each archive policy will have a different value for granularity. If a threshold alarm time interval is shorter than the granularity for that specific metric, the alarm will remain in an **insufficient_data** state.

Metric Commands
^^^^^^^^^^^^^^^

We can list the metrics in our project using the command:

.. code-block:: bash

  openstack metric list

  #Example Output

  +--------------------------------------+---------------------+-------------------------------+---------+--------------------------------------+
  | id                                   | archive_policy/name | name                          | unit    | resource_id                          |
  +--------------------------------------+---------------------+-------------------------------+---------+--------------------------------------+
  | 0009ecb6-ffdc-4b62-a870-10eee6be7c93 | ceilometer-low      | disk.root.size                | GB      | 1f2dbe24-1011-4039-8102-405d494eb14d |
  | 00402d05-6af5-43c1-a1e9-03806ff04c3b | ceilometer-low-rate | disk.device.write.requests    | request | 136027f2-664f-5fb5-ba20-68800504f3f7 |
  | 00556936-8bd6-42f3-9e8f-17cb43824778 | ceilometer-low      | disk.root.size                | GB      | 8367a387-65e0-4071-b0ea-e5f434cc74ed |
  | 006d7503-fb1a-4c66-91e1-0f2345b9cf92 | ceilometer-low      | disk.root.size                | GB      | d1a8a32a-6904-4545-8ad4-7cde34101b61 |
  | 006e123e-18a7-4ce4-9a10-81b778399962 | ceilometer-low      | memory.usage                  | MB      | f4ba2800-93c3-4bdf-b010-c9f5994e62aa |
  | 00815878-49ed-4e6a-9884-7556133306f1 | ceilometer-low-rate | network.incoming.packets      | packet  | 126e2936-7acd-5796-aee1-34946379a2de |
  | 008745cc-6c9b-41de-a09b-5b731c2d4ab1 | ceilometer-low      | memory                        | MB      | ce9638ca-c5b4-4824-b930-0d912a583f0b |
  | 009fc9c6-9896-4388-9214-61c7ff332c53 | ceilometer-low      | vcpus                         | vcpu    | 8808e24e-03bb-4e1b-aa28-d1c393f5e935 |
  | 00b46973-0df5-4a3c-b9d5-795a14cae0b8 | ceilometer-low      | disk.root.size                | GB      | 564973c9-5107-4851-a776-02156ff6f78a |
  | 00ba0c03-89fe-4406-90b5-88fe94194c87 | ceilometer-low      | memory.usage                  | MB      | dc5024b1-cd1a-4f25-99e2-a17471e15530 |
  | 00c212c7-93a9-461b-a657-556181c9b4c1 | ceilometer-low      | compute.instance.booting.time | sec     | cc90947e-cad8-4890-a912-81459f718be0 |
  |     ............................     |  ...............    |    .......................    |  .....  |    ..............................    |

  # Note: This will return a list of every metric for every resource in the project

  
To view more information about the metric, we can use:

.. code-block:: bash

  openstack metric show <uuid>

  # Example output
  +--------------------------------+-------------------------------------------------------------------+
  | Field                          | Value                                                             |
  +--------------------------------+-------------------------------------------------------------------+
  | archive_policy/name            | ceilometer-low                                                    |
  | creator                        | e764d5abc65843fcb3bb060c80169871:4de86830e89b4a46b590536571b6ccd4 |
  | id                             | f247f0ed-e5f0-4b72-95cc-b7771f984e83                              |
  | name                           | memory                                                            |
  | resource/created_by_project_id | 4de86830e89b4a46b590536571b6ccd4                                  |
  | resource/created_by_user_id    | e764d5abc65843fcb3bb060c80169871                                  |
  | resource/creator               | e764d5abc65843fcb3bb060c80169871:4de86830e89b4a46b590536571b6ccd4 |
  | resource/ended_at              | None                                                              |
  | resource/id                    | 69252292-8a40-400b-9446-8c1bfa9f471d                              |
  | resource/original_resource_id  | 69252292-8a40-400b-9446-8c1bfa9f471d                              |
  | resource/project_id            | 6a2f34e232744e59a5af8e105507f076                                  |
  | resource/revision_end          | None                                                              |
  | resource/revision_start        | 2020-10-13T14:01:40.979875+00:00                                  |
  | resource/started_at            | 2020-08-06T13:48:43.421894+00:00                                  |
  | resource/type                  | instance                                                          |
  | resource/user_id               | 569d4d38222c86c68585e194b200eddea857137476dc76360b546b48f4319dde  |
  | unit                           | MB                                                                |
  +--------------------------------+-------------------------------------------------------------------+


We can view the metric resource list as well. The following command will return a table containing every single resource (instance, instance disk, etc) which metrics are attached to.

.. code-block:: bash

  openstack metric resource show <resource-id>
  # This will return every single resource in the project, this includes the volumes, disks etc associated to each VM


To view the measurements of a metric, we can use the command:

.. code-block:: bash

  openstack metric measures show <metric-id>


To view the resource which has a metric linked to it, we can use the command:

.. code-block:: bash

  openstack metric resource show <resource-id>

  # This will also show the IDs for the metrics attached to that specific resource
  +-----------------------+---------------------------------------------------------------------+
  | Field                 | Value                                                               |
  +-----------------------+---------------------------------------------------------------------+
  | created_by_project_id | 4de86830e89b4a46b590536571b6ccd4                                    |
  | created_by_user_id    | e764d5abc65843fcb3bb060c80169871                                    |
  | creator               | e764d5abc65843fcb3bb060c80169871:4de86830e89b4a46b590536571b6ccd4   |
  | ended_at              | 2020-09-28T13:00:17.994533+00:00                                    |
  | id                    | 17ae2b28-7ed1-43e7-9099-e7e1134a10ad                                |
  | metrics               | compute.instance.booting.time: 8db0701c-1e3e-4af0-95b2-dc95cf1010c0 |
  |                       | cpu: f5d3c5ca-0f03-49b4-9ec0-e25d50cd7abd                           |
  |                       | disk.ephemeral.size: 0b813829-7d48-4e96-a88c-472dc739b427           |
  |                       | disk.root.size: d8309af3-ef3d-4043-b80d-7f88b0b10d57                |
  |                       | memory.usage: 0a994a4c-7d9b-45d0-8e1f-82a9d6004b3e                  |
  |                       | memory: 4e4b3247-2197-441f-a880-4af6edc89747                        |
  |                       | vcpus: 65799f16-836d-445e-a395-d0bb6ac56ba5                         |
  | original_resource_id  | 17ae2b28-7ed1-43e7-9099-e7e1134a10ad                                |
  | project_id            | PROJECT_ID                                                          |
  | revision_end          | None                                                                |
  | revision_start        | 2020-09-28T13:00:31.836563+00:00                                    |
  | started_at            | 2020-09-28T12:52:51.968895+00:00                                    |
  | type                  | instance                                                            |
  | user_id               | USER_ID                                                             |
  +-----------------------+---------------------------------------------------------------------+



To view the measurements of a metric, we can use the command:

.. code-block:: bash

  openstack metric measures show <metric-id>

  #Example Output

  +---------------------------+-------------+--------+
  | timestamp                 | granularity |  value |
  +---------------------------+-------------+--------+
  | 2020-09-13T16:00:00+01:00 |       300.0 | 8192.0 |
  | 2020-09-13T17:00:00+01:00 |       300.0 | 8192.0 |
  | 2020-09-13T18:00:00+01:00 |       300.0 | 8192.0 |
  | 2020-09-13T19:00:00+01:00 |       300.0 | 8192.0 |
  | 2020-09-13T20:00:00+01:00 |       300.0 | 8192.0 |
  | 2020-09-13T21:00:00+01:00 |       300.0 | 8192.0 |
  | 2020-09-13T22:00:00+01:00 |       300.0 | 8192.0 |
  | 2020-09-13T23:00:00+01:00 |       300.0 | 8192.0 |
  | 2020-09-14T00:00:00+01:00 |       300.0 | 8192.0 |
  |           ...             |    ...      |  ...   |

  #Note: This will return ALL measurements for the metric!


To view the list of resource types:

..code-block:: bash

  openstack metric resource-type list


References
----------

Gnocchi Documentation: https://gnocchi.xyz/stable_4.2/rest.html

Telemetry Measurements for Train: https://docs.openstack.org/ceilometer/train/admin/telemetry-measurements.html

Gnocchi Aggregation: https://medium.com/@berndbausch/how-i-learned-to-stop-worrying-and-love-gnocchi-aggregation-c98dfa2e20fe

Gnocchi Glossary: https://gnocchi.xyz/stable_4.2/glossary.html

Aodh Alarms: https://docs.openstack.org/aodh/train/admin/telemetry-alarms.html
