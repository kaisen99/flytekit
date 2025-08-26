# VersionParameters

This class encapsulates parameters utilized in version hash generation. It accepts a function and/or a container image to determine the version. The class supports optional parameters for specifying a pod template or its name.

## Attributes

- **func**: Callable[P, FuncOut]
  - The function to generate a version for. This is an optional parameter and can be any callable that matches the specified parameter and return types.

- **container_image**: Optional[[Union](flytekit_models_literals_union)[str, [ImageSpec](flytekit_image_spec_image_spec_imagespec)]] = None
  - The container image to generate a version for. This can be a string representing the image name or an ImageSpec object.

- **pod_template**: Optional[[PodTemplate](flytekit_core_pod_template_podtemplate)] = None
  - None

- **pod_template_name**: Optional[str] = None
  - None

## Constructors
def VersionParameters(func: Callable[P, FuncOut], container_image: Optional[[Union](flytekit_models_literals_union)[str, [ImageSpec](flytekit_image_spec_image_spec_imagespec)]] = None)
-  Parameters used for version hash generation.
- **Parameters**

  - **func**: Callable[P, FuncOut]
    - The function to generate a version for. This is an optional parameter and can be any callable
                 that matches the specified parameter and return types.
  - **container_image**: Optional[[Union](flytekit_models_literals_union)[str, [ImageSpec](flytekit_image_spec_image_spec_imagespec)]]
    - The container image to generate a version for. This can be a string representing the
                            image name or an ImageSpec object.



