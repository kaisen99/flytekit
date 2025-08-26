# SlurmJobMetadata

This class represents metadata associated with a Slurm job. It provides access to job-specific information and facilitates SSH client connections. The class utilizes a job ID and SSH configuration for establishing connections.

## Attributes

- **job_id**: str
  - Slurm job id.

- **ssh_config**: Dict[str, [Union](flytekit_models_literals_union)[str, List[str], Tuple[str, ...]]]
  - SSH configuration options for establishing client connections.

## Constructors
def SlurmJobMetadata(job_id: str, ssh_config: Dict[str, [Union](flytekit_models_literals_union)[str, List[str], Tuple[str, ...]]])
-  Initializes SlurmJobMetadata.
- **Parameters**

  - **job_id**: str
    - Slurm job id.
  - **ssh_config**: Dict[str, [Union](flytekit_models_literals_union)[str, List[str], Tuple[str, ...]]]
    - Options of SSH client connection. For available options, please refer to the ssh_utils module.



