# RunPolicy

This class defines the execution policies for a Kubeflow job, controlling aspects like pod cleanup and job duration. It allows configuration of pod cleanup behavior, time-to-live after completion, and active deadline. The class provides a way to manage job retries through the backoff limit.

## Attributes

- **clean_pod_policy**: [CleanPodPolicy](flytekitplugins_kftensorflow_task_cleanpodpolicy) = None
  - The policy for cleaning up pods after the job completes.

- **ttl_seconds_after_finished**: Optional[int] = None
  - The time-to-live (TTL) in seconds for cleaning up finished jobs.

- **active_deadline_seconds**: Optional[int] = None
  - The duration (in seconds) since startTime during which the job can remain active before it is terminated. Must be a positive integer. This setting applies only to pods where restartPolicy is OnFailure or Always.

- **backoff_limit**: Optional[int] = None
  - The number of retries before marking this job as failed.

## Constructors
def RunPolicy(clean_pod_policy: [CleanPodPolicy](flytekitplugins_kftensorflow_task_cleanpodpolicy) = None, ttl_seconds_after_finished: Optional[int] = None, active_deadline_seconds: Optional[int] = None, backoff_limit: Optional[int] = None)
-  RunPolicy describes a set of policies to apply to the execution of a Kubeflow job.
- **Parameters**

  - **clean_pod_policy**: [CleanPodPolicy](flytekitplugins_kftensorflow_task_cleanpodpolicy)
    - The policy for cleaning up pods after the job completes.
  - **ttl_seconds_after_finished**: Optional[int]
    - The time-to-live (TTL) in seconds for cleaning up finished jobs.
  - **active_deadline_seconds**: Optional[int]
    - The duration (in seconds) since startTime during which the job can remain active before it is terminated. Must be a positive integer. This setting applies only to pods where restartPolicy is OnFailure or Always.
  - **backoff_limit**: Optional[int]
    - The number of retries before marking this job as failed.



