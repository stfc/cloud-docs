=========
Gnocchi
=========

Gnocchi processes data from Ceilometer and stores the processed data which can be accessed by users.
It correlates measurements from Ceilometer to **resources** and **metrics**.

**Metric:** A resource property which can be measured. For example the CPU, memory usage etc.

**Resource:** These can be instances, volumes, networks etc.


gnocchiclient
#############

The gnocchi command line tool can be installed using pip:

.. code-block:: bash

  pip install gnocchiclient

This provides the following Openstack commands:

.. code-block:: bash

  openstack [metric-command]

    metric aggregates
    metric archive-policy create
    metric archive-policy delete
    metric archive-policy list
    metric archive-policy show
    metric archive-policy update
    metric archive-policy-rule create
    metric archive-policy-rule delete
    metric archive-policy-rule list
    metric archive-policy-rule show
    metric benchmark measures add
    metric benchmark measures show
    metric benchmark metric create
    metric benchmark metric show
    metric capabilities list
    metric create
    metric delete
    metric list
    metric measures add
    metric measures aggregation
    metric measures batch-metrics
    metric measures batch-resources-metrics
    metric measures show
    metric resource batch delete
    metric resource create
    metric resource delete
    metric resource history
    metric resource list
    metric resource search
    metric resource show
    metric resource update
    metric resource-type create
    metric resource-type delete
    metric resource-type list
    metric resource-type show
    metric resource-type update
    metric server version
    metric show
    metric status



Metrics
#######

Metrics consist of:
  - Name: unique name for the metric
  - Unit (e.g. MB, ns, B/s)
  - Resource ID - where the measurement is from
  - Unique User ID (UUID): Identifies the metric
  - Archive policy: stores and aggregates the measures

Archive policies determine how long aggregates are kept and how they are aggregated.
The archive policies can be seen using the command:

.. code-block:: bash

  openstack metric archive-policy list

  +----------------------+-------------+-----------------------------------------+---------------------+
  | name                 | back_window | definition                              | aggregation_methods |
  +----------------------+-------------+-----------------------------------------+---------------------+
  | ceilometer-high      |           0 | - points: 3600, timespan: 1:00:00,      | mean                |
  |                      |             | granularity: 0:00:01                    |                     |
  |                      |             | - points: 1440, timespan: 1 day,        |                     |
  |                      |             | 0:00:00, granularity: 0:01:00           |                     |
  |                      |             | - points: 8760, timespan: 365 days,     |                     |
  |                      |             | 0:00:00, granularity: 1:00:00           |                     |
  | ceilometer-high-rate |           0 | - points: 3600, timespan: 1:00:00,      | rate:mean, mean     |
  |                      |             | granularity: 0:00:01                    |                     |
  |                      |             | - points: 1440, timespan: 1 day,        |                     |
  |                      |             | 0:00:00, granularity: 0:01:00           |                     |
  |                      |             | - points: 8760, timespan: 365 days,     |                     |
  |                      |             | 0:00:00, granularity: 1:00:00           |                     |
  | ceilometer-low       |           0 | - points: 8640, timespan: 30 days,      | mean                |
  |                      |             | 0:00:00, granularity: 0:05:00           |                     |
  | ceilometer-low-rate  |           0 | - points: 8640, timespan: 30 days,      | rate:mean, mean     |
  |                      |             | 0:00:00, granularity: 0:05:00           |                     |
  +----------------------+-------------+-----------------------------------------+---------------------+

This table shows each archive-policy and how the raw datapoints for each metric is stored.
As an example, let's view the detail for one of the archive policies.

To view an archive policy, use the command:

.. code-block:: bash

  openstack metric archive-policy show <archive-policy>


For example:

.. code-block:: bash

  openstack archive-policy show  ceilometer-high

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

For the archive policy ceilometer-high:

- The **mean** is stored for each interval.
- Stores **one hour** of data in **one second** intervals. (3600 data points)
- Stores **one day**  of data in **one minute** intervals. (1440 data points)
- Stores **one year** of data in **one hour** intervals. (8760 data points)


To view the list of metrics:

.. code-block:: bash

  openstack metric list

This will show the metrics which have been created/visible to the user.

To view the metric resources:

.. code-block:: bash

  openstack metric resource list


To view the properties of a specific metric:

.. code-block:: bash

  openstack metric show <metric-id>

Metrics can be created by the user using the gnocchiclient plugin in the Openstack CLI:

.. code-block:: bash

  openstack metric create [-h] [--resource-id RESOURCE_ID] [-f {json,shell,table,value,yaml}]
                               [-c COLUMN] [--noindent] [--prefix PREFIX] [--max-width <integer>]
                               [--fit-width] [--print-empty]
                               [--archive-policy-name ARCHIVE_POLICY_NAME] [--unit UNIT]
                               [METRIC_NAME]

**Note:** Once a metric has been created, the archive policy attribute for that metric is fixed and cannot be changed.



References:
###########

Gnocchi Documentation: https://gnocchi.xyz/stable_4.2/rest.html

Telemetry Measurements for Train: https://docs.openstack.org/ceilometer/train/admin/telemetry-measurements.html

Gnocchi Aggregation: https://medium.com/@berndbausch/how-i-learned-to-stop-worrying-and-love-gnocchi-aggregation-c98dfa2e20fe

Gnocchi Glossary: https://gnocchi.xyz/stable_4.2/glossary.html
