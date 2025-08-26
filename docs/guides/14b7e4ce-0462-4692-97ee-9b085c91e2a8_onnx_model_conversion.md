
<!--
help_text: ''
key: summary_onnx_model_conversion_69b2d786-7b69-44d1-b3dd-865674c69acd
modules:
- flytekitplugins.onnxpytorch.schema
- flytekitplugins.onnxscikitlearn.schema
- flytekitplugins.onnxtensorflow.schema
questions_to_answer: []
type: summary

-->
ONNX Model Conversion

The ONNX model conversion feature enables seamless transformation of machine learning models from popular frameworks like PyTorch, Scikit-learn, and TensorFlow into the Open Neural Network Exchange (ONNX) format. This capability is integrated directly into the platform's type system, allowing models to be converted and managed as first-class data types within workflows.

Converting models to ONNX provides several benefits, including:
*   **Interoperability**: Deploy models across various hardware and software platforms that support ONNX Runtime.
*   **Optimization**: Leverage ONNX Runtime's optimizations for improved inference performance.
*   **Portability**: Easily move models between different frameworks for deployment or further development.

### Core Concepts

Model conversion is handled through a set of specialized types and transformers that integrate with the platform's data serialization mechanisms.

#### Model Wrappers

To facilitate conversion, models from supported frameworks are wrapped in specific data classes:

*   **PyTorch Models**: Encapsulated by `PyTorch2ONNX`. This class holds a PyTorch model, which can be a `torch.nn.Module`, `torch.jit.ScriptModule`, or `torch.jit.ScriptFunction`.
*   **Scikit-learn Models**: Encapsulated by `ScikitLearn2ONNX`. This class holds a Scikit-learn model, typically an instance of `sklearn.base.BaseEstimator`.
*   **TensorFlow Models**: Encapsulated by `TensorFlow2ONNX`. This class holds a TensorFlow Keras model, specifically `tf.keras.Model`.

These wrapper classes serve as the input types for tasks that produce models intended for ONNX conversion.

#### Conversion Configurations

Each framework's conversion process requires specific parameters. These are defined in dedicated configuration classes:

*   **PyTorch Conversion Configuration**: `PyTorch2ONNXConfig` specifies parameters for converting PyTorch models. Key parameters include:
    *   `args`: The input to the model, crucial for tracing the graph. This can be a `Tuple` of tensors or a single `torch.Tensor`.
    *   `opset_version`: The ONNX opset version to target.
    *   `input_names`, `output_names`: Names for the input and output nodes in the ONNX graph.
    *   `dynamic_axes`: Defines dynamic axes for inputs/outputs, enabling flexible batch sizes or input dimensions.
    *   `export_params`, `verbose`, `training`, `do_constant_folding`: Control various aspects of the export process.
*   **Scikit-learn Conversion Configuration**: `ScikitLearn2ONNXConfig` defines parameters for converting Scikit-learn models. Important parameters include:
    *   `initial_types`: A list of tuples specifying the names and types of the model's inputs. This is critical for `skl2onnx` to correctly infer the graph. Types must be from `skl2onnx.common.data_types`.
    *   `target_opset`: The ONNX opset number.
    *   `name`, `doc_string`: Metadata for the ONNX model.
    *   `custom_conversion_functions`, `custom_shape_calculators`, `custom_parsers`: Allow for extending conversion logic for custom Scikit-learn estimators.
*   **TensorFlow Conversion Configuration**: `TensorFlow2ONNXConfig` provides parameters for converting TensorFlow models. Key parameters include:
    *   `input_signature`: The shape and data type of the model's inputs, typically a `tf.TensorSpec` or `np.ndarray`. This is essential for tracing the TensorFlow graph.
    *   `opset`, `extra_opset`: The ONNX opset versions to use.
    *   `custom_ops`, `custom_op_handlers`, `custom_rewriter`: Support for handling custom TensorFlow operations.
    *   `shape_override`: Overrides input shapes provided by TensorFlow.
    *   `large_model`: Enables the ONNX external tensor storage format for large models.

These configuration classes are typically associated with the model wrapper types using the platform's `with_config` mechanism.

#### Type Transformers

The actual conversion logic is encapsulated within `TypeTransformer` implementations: `PyTorch2ONNXTransformer`, `ScikitLearn2ONNXTransformer`, and `TensorFlow2ONNXTransformer`. These transformers automatically handle the serialization of the model object into an ONNX file and its subsequent upload to remote storage when a task returns a model wrapper type (e.g., `PyTorch2ONNX`).

When a task returns a model wrapper type with an associated configuration, the transformer:
1.  Retrieves the model instance and the conversion configuration.
2.  Invokes an internal `to_onnx` function, passing the model and configuration parameters. This function performs the framework-specific conversion.
3.  Uploads the resulting ONNX file to the platform's file access system.
4.  Returns a `Literal` representing the ONNX file's URI, formatted as an ONNX blob.

Conversely, when an ONNX blob is passed as an input to a task, the transformer downloads the ONNX file and provides it as an `ONNXFile` object, which is a simple wrapper around the local path to the ONNX model.

### Supported Frameworks and Usage

To convert a model, define a task that returns the appropriate model wrapper type, and attach the conversion configuration using `with_config`.

#### PyTorch to ONNX

To convert a PyTorch model, return an instance of `PyTorch2ONNX` from your task, configured with `PyTorch2ONNXConfig`.

```python
import torch
import torch.nn as nn
from flytekit import task, workflow
from flytekitplugins.onnxpytorch.schema import PyTorch2ONNX, PyTorch2ONNXConfig

# Define a simple PyTorch model
class SimpleModel(nn.Module):
    def __init__(self):
        super().__init__()
        self.linear = nn.Linear(10, 1)

    def forward(self, x):
        return self.linear(x)

@task
def convert_pytorch_model() -> PyTorch2ONNX:
    model = SimpleModel()
    # Define a dummy input for tracing
    dummy_input = torch.randn(1, 10)

    # Configure the ONNX conversion
    onnx_config = PyTorch2ONNXConfig(
        args=(dummy_input,),
        opset_version=11,
        input_names=["input"],
        output_names=["output"],
        dynamic_axes={"input": {0: "batch_size"}, "output": {0: "batch_size"}},
    )

    # Return the model wrapped with its configuration
    return PyTorch2ONNX(model=model).with_config(onnx_config)

@task
def use_onnx_model(onnx_model_file: PyTorch2ONNX):
    # onnx_model_file will be an ONNXFile object, containing the path to the ONNX model
    print(f"Received ONNX model at: {onnx_model_file.path}")
    # You can now load and use the ONNX model with ONNX Runtime
    # import onnxruntime as rt
    # sess = rt.InferenceSession(onnx_model_file.path)
    # ...

@workflow
def pytorch_onnx_workflow():
    onnx_model = convert_pytorch_model()
    use_onnx_model(onnx_model_file=onnx_model)
```

#### Scikit-learn to ONNX

For Scikit-learn models, return an instance of `ScikitLearn2ONNX` from your task, configured with `ScikitLearn2ONNXConfig`. Ensure `initial_types` are correctly specified using types from `skl2onnx.common.data_types`.

```python
import numpy as np
from sklearn.linear_model import LogisticRegression
from flytekit import task, workflow
from flytekitplugins.onnxscikitlearn.schema import ScikitLearn2ONNX, ScikitLearn2ONNXConfig
from skl2onnx.common.data_types import FloatTensorType

@task
def convert_sklearn_model() -> ScikitLearn2ONNX:
    # Train a simple Scikit-learn model
    X = np.array([[0, 0], [0, 1], [1, 0], [1, 1]], dtype=np.float32)
    y = np.array([0, 0, 1, 1])
    model = LogisticRegression().fit(X, y)

    # Define initial types for ONNX conversion
    # The input is a float tensor with a dynamic batch size and 2 features
    initial_type = [("input", FloatTensorType([None, 2]))]

    # Configure the ONNX conversion
    onnx_config = ScikitLearn2ONNXConfig(
        initial_types=initial_type,
        target_opset=11,
        name="LogisticRegressionModel",
    )

    # Return the model wrapped with its configuration
    return ScikitLearn2ONNX(model=model).with_config(onnx_config)

@task
def use_onnx_model_sklearn(onnx_model_file: ScikitLearn2ONNX):
    print(f"Received ONNX model at: {onnx_model_file.path}")
    # Load and use with ONNX Runtime
    # import onnxruntime as rt
    # sess = rt.InferenceSession(onnx_model_file.path)
    # ...

@workflow
def sklearn_onnx_workflow():
    onnx_model = convert_sklearn_model()
    use_onnx_model_sklearn(onnx_model_file=onnx_model)
```

#### TensorFlow to ONNX

For TensorFlow Keras models, return an instance of `TensorFlow2ONNX` from your task, configured with `TensorFlow2ONNXConfig`. The `input_signature` is crucial for successful conversion.

```python
import tensorflow as tf
import numpy as np
from flytekit import task, workflow
from flytekitplugins.onnxtensorflow.schema import TensorFlow2ONNX, TensorFlow2ONNXConfig

@task
def convert_tensorflow_model() -> TensorFlow2ONNX:
    # Define a simple TensorFlow Keras model
    model = tf.keras.Sequential([
        tf.keras.layers.Input(shape=(10,), dtype=tf.float32, name="input_layer"),
        tf.keras.layers.Dense(1, activation='sigmoid')
    ])
    model.compile(optimizer='adam', loss='binary_crossentropy')

    # Define the input signature for ONNX conversion
    input_signature = tf.TensorSpec(shape=[None, 10], dtype=tf.float32, name="input_tensor")

    # Configure the ONNX conversion
    onnx_config = TensorFlow2ONNXConfig(
        input_signature=input_signature,
        opset=11,
    )

    # Return the model wrapped with its configuration
    return TensorFlow2ONNX(model=model).with_config(onnx_config)

@task
def use_onnx_model_tf(onnx_model_file: TensorFlow2ONNX):
    print(f"Received ONNX model at: {onnx_model_file.path}")
    # Load and use with ONNX Runtime
    # import onnxruntime as rt
    # sess = rt.InferenceSession(onnx_model_file.path)
    # ...

@workflow
def tensorflow_onnx_workflow():
    onnx_model = convert_tensorflow_model()
    use_onnx_model_tf(onnx_model_file=onnx_model)
```

### Retrieving ONNX Models

When an ONNX model is returned from a task, it is stored as a blob. In subsequent tasks that consume this ONNX model, the platform automatically downloads the file and provides it as an `ONNXFile` object. This object contains a `path` attribute, which is the local file path to the downloaded ONNX model. You can then use this path with ONNX Runtime or other ONNX-compatible libraries.

```python
from flytekitplugins.onnxpytorch.schema import PyTorch2ONNX
from flytekit.types.file import ONNXFile # This is the type received by downstream tasks

@task
def process_onnx_model(model_file: ONNXFile):
    """
    This task receives the ONNX model as a local file.
    """
    print(f"Processing ONNX model from: {model_file.path}")
    # Example: Load the ONNX model using onnxruntime
    # import onnxruntime as rt
    # sess = rt.InferenceSession(model_file.path)
    # input_name = sess.get_inputs()[0].name
    # output_name = sess.get_outputs()[0].name
    # dummy_input = np.random.rand(1, 10).astype(np.float32)
    # result = sess.run([output_name], {input_name: dummy_input})
    # print(f"Inference result: {result}")
```

### Best Practices and Considerations

*   **Input Signatures/Arguments**: Providing accurate input signatures (`input_signature` for TensorFlow, `args` for PyTorch, `initial_types` for Scikit-learn) is paramount. These define the shape and type of data the model expects, which is essential for the conversion tool to correctly trace the model's graph. Incorrect specifications can lead to conversion failures or models that do not behave as expected.
*   **Opset Version**: Specify an `opset_version` that is compatible with your target ONNX Runtime environment. Newer opsets may support more operations but might not be universally supported by older runtimes.
*   **Dynamic Axes**: For models that need to handle variable batch sizes or input dimensions, use `dynamic_axes` (PyTorch) or define dynamic dimensions in `input_signature`/`initial_types`.
*   **Custom Operations**: If your model uses custom operations not natively supported by the ONNX converters, explore the `custom_ops`, `custom_op_handlers`, `custom_conversion_functions`, or `custom_parsers` options provided by the respective configuration classes. This allows you to define how these custom operations should be represented in ONNX.
*   **Error Handling**: Conversion can fail if the model or configuration is invalid. Ensure your tasks handle potential `TypeTransformerFailedError` exceptions during development and testing.
*   **Model Size**: For very large models, consider using the `large_model` option in `TensorFlow2ONNXConfig` to enable ONNX external tensor storage, which stores large tensors in separate files.
*   **Dependencies**: Ensure that the necessary framework-specific ONNX conversion libraries (e.g., `torch.onnx`, `skl2onnx`, `tf2onnx`) are installed in your task's execution environment. The `ScikitLearn2ONNXConfig` specifically validates `initial_types` against `skl2onnx.common.data_types`, highlighting this dependency.
<!--
key: summary_onnx_model_conversion_69b2d786-7b69-44d1-b3dd-865674c69acd
type: summary_end

-->
<!--
code_unit: flytekitplugins.onnxpytorch.examples.pytorch_to_onnx
code_unit_type: class
help_text: ''
key: example_1abe1d47-ee30-474d-b3f7-9a113a167afc
type: example

-->
<!--
code_unit: flytekitplugins.onnxscikitlearn.examples.sklearn_to_onnx
code_unit_type: class
help_text: ''
key: example_180b5bd7-0712-4174-9dc8-31a23c02a928
type: example

-->
<!--
code_unit: flytekitplugins.onnxtensorflow.examples.tensorflow_to_onnx
code_unit_type: class
help_text: ''
key: example_4ae65654-5cc1-4f5a-a752-90ff46aaa812
type: example

-->