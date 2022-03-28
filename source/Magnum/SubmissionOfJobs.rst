========================================
Submitting jobs to a Kubernetes Cluster
========================================

    **Important:** Magnum is currently in technical preview on STFC Cloud. If you have any feedback or suggestions please send it to cloud-support@gridpp.rl.ac.uk

A Kubernetes job creates one or more pods on a cluster and have the added benefit of being retried until a specified number of pods successfully terminate. Jobs
are described by YAML and can be executed using `kubectl`.

Submitting jobs
----------------------------------

Jobs are defined by a YAML config with a `kind` parameter of Job. Below is an example job config for computing :math:`\pi` to 2000 places.

.. code-block:: YAML
    :caption: job.yaml

    apiVersion: batch/v1
    kind: Job
    metadata:
      name: pi
      spec:
      template:
        spec:
          containers:
          - name: pi
            #Docker image
            image: perl
            #Compute pi to 2000 places by running this command in the container
            command: ["perl",  "-Mbignum=bpi", "-wle", "print bpi(2000)"]
          restartPolicy: Never
      #Number of retries to attempt before stopping (default is 6)
      backoffLimit: 4

To run this job use:

.. code-block:: bash

    kubectl apply -f ./job.yaml

This will result in the creation of a pod with a single container named `pi` that is based on the `perl` image. The specified command, equivalent to
``perl -Mbignum=bpi -wle "print bpi(2000)"``, will then be executed in the container. You can check on the status of the job using

.. code-block:: bash

    kubectl describe jobs/pi

Then the output from the container can be obtained through

.. code-block:: bash

    pods=$(kubectl get pods --selector=job-name=pi --output=jsonpath='{.items[*].metadata.name}')
    kubectl logs $pods

.. Important::
    The job and pods it creates are usually kept after execution to allow the logs to be seen, these can be deleted using
    ``kubectl delete jobs/pi`` or ``kubectl delete -f ./job.yaml``.

    One way to avoid a build up of undeleted jobs and pods is to use `ttlSecondsAfterFinished` which was delcared stable as of Kubernetes v1.23.
    This defines the time after which a job in a Complete/Failed state should be deleted automatically. For example the following job will be
    deleted 100 seconds after finishing.

    .. code-block:: YAML

        apiVersion: batch/v1
        kind: Job
        metadata:
        name: pi
        spec:
          #Delete the job 100 seconds after successful completion
          ttlSecondsAfterFinished: 100
          template:
            spec:
              containers:
              - name: pi
                image: perl
                command: ["perl",  "-Mbignum=bpi", "-wle", "print bpi(2000)"]
              restartPolicy: Never

Parallel execution
----------------------------------

The above example runs a single pod until completion or 4 successive failures. It is also possible to execute multiple instances of pods in parallel. For a simple example we can require
a `fixed completion count` by assigning ``.spec.completions`` in the YAML file to require more than one successful execution of a pod is required before the job is considered complete.
We can then also specify ``.spec.parallelism`` to increase the number of pods that can be running at any one time. For example, the below will run up to 2 pods in parallel until
8 of them finish successfully.

    .. code-block:: YAML

        apiVersion: batch/v1
        kind: Job
        metadata:
        name: pi
        spec:
          ttlSecondsAfterFinished: 100
          #Require 8 successful completions before the job is considered finished
          completions: 8
          #Allow up to 2 pods to be running in parallel
          parallelism: 2
          template:
            spec:
              containers:
              - name: pi
                image: perl
                command: ["perl",  "-Mbignum=bpi", "-wle", "print bpi(2000)"]
              restartPolicy: Never

If one pod fails a new pod will be created to take its place and the job will continue. 

You can also use a `work queue` for parallel jobs by not specifying ``.spec.completions`` at all. In this case the pods should coordinate amongst themselves or via an external service
to determine when they have finished as when any one of them successfully exits the job will be considered complete. Therefore each should exit only there is no more work for `any` of the pods.

Scheduling jobs
----------------------------------
To run jobs on a schedule you can use CronJobs. These are also described using a YAML file, for example:

.. code-block:: YAML
    :caption: cronjob.yaml

    apiVersion: batch/v1
    kind: CronJob
    metadata:
      name: hello
    spec:
      #This described when the job described below should be executed
      schedule: "* * * * *"
      jobTemplate:
        spec:
          template:
            spec:
              containers:
              - name: hello
                image: busybox:1.28
                imagePullPolicy: IfNotPresent
                command: ["/bin/sh", "-c", "date; echo Hello from the Kubernetes cluster"]
              restartPolicy: OnFailure

This will run a job every minute that prints  "Hello from the Kubernetes cluster". You can create the CronJob using:

.. code-block:: bash

   kubectl create -f ./cronjob.yaml

You may check on the status of the CronJob using ``kubectl get cronjob hello`` and watch the jobs it creates in real time using ``kubectl get jobs --watch``. From the latter
you will see that the job names appear as `hello-` followed by some numbers e.g. `hello-27474266`. This can be used to view the output of the job using

.. code-block:: bash

    pods=$(kubectl get pods --selector=job-name=hello-27474266 --output=jsonpath='{.items[*].metadata.name}')
    kubectl logs $pods

The `schedule` parameter, which in this case causes the job to run every minute, is in the following format.

.. code-block:: bash

    minute (0 - 59), hour (0 - 23), day of the month(1 - 31), month (1 - 12), day of the week (0 - 6)

Where the day of the week is 0 for Sunday and 6 for Saturday. In the example the asterisk is used to indicate `any`. You may find tools such as https://crontab.guru/ helpful
in writing these. For example ``5 4 * * 2`` will run at 04:05 on Tuesdays. The time specified is the local time of the machine.

You may delete the CronJob, along with any of its existing jobs and pods using

.. code-block:: bash

   kubectl delete cronjob hello


References
----------

https://kubernetes.io/docs/concepts/workloads/controllers/job/

https://kubernetes.io/docs/tasks/job/automated-tasks-with-cron-jobs/
