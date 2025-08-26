# CleanPodPolicy

This class, CleanPodPolicy, defines the strategies for handling pods upon job completion. It provides an enumeration of policies, including options to retain no pods, clean all pods, or only clean running pods. This class is essential for managing pod lifecycles within a job execution framework.

## Attributes

- **NONE**: Enum = kubeflow_common.CLEANPOD_POLICY_NONE
  - CleanPodPolicy describes how to deal with pods when the job is finished.

- **ALL**: Enum = kubeflow_common.CLEANPOD_POLICY_ALL
  - CleanPodPolicy describes how to deal with pods when the job is finished.

- **RUNNING**: Enum = kubeflow_common.CLEANPOD_POLICY_RUNNING
  - CleanPodPolicy describes how to deal with pods when the job is finished.



