
<!--
help_text: ''
key: summary_omegaconf_integration_7275b047-1750-4d4d-ba57-9e9e87dd1657
modules:
- flytekitplugins.omegaconf
- flytekitplugins.omegaconf.listconfig_transformer
- flytekitplugins.omegaconf.type_information
- flytekitplugins.omegaconf.config
- flytekitplugins.omegaconf.dictconfig_transformer
questions_to_answer: []
type: summary

-->
Flytekit's OmegaConf integration enables seamless handling of `omegaconf.DictConfig` and `omegaconf.ListConfig` objects as inputs and outputs for Flyte tasks and workflows. This allows developers to leverage OmegaConf's powerful hierarchical configuration capabilities directly within their Flyte pipelines, promoting robust and flexible configuration management.

### Core Capabilities

The integration provides custom type transformers that convert OmegaConf objects to and from Flyte's internal literal representation.

*   **`DictConfig` Support:** The `DictConfigTransformer` handles the serialization and deserialization of `omegaconf.DictConfig` objects. It converts `DictConfig` instances into Flyte's `LiteralType(simple=SimpleType.STRUCT)` and reconstructs them upon deserialization. This transformer is designed to preserve the structure and type information, including any underlying structured dataclass types.
*   **`ListConfig` Support:** The `ListConfigTransformer` manages `omegaconf.ListConfig` objects. Similar to `DictConfig`, it uses `LiteralType(simple=SimpleType.STRUCT)` to represent lists. A key feature is its ability to handle heterogeneous lists (lists containing elements of different types) and `omegaconf.MISSING` values by embedding type information for each element directly within the serialized `STRUCT`.

### Deserialization Modes for `DictConfig`

The `OmegaConfTransformerMode` enum, located in `flytekitplugins.omegaconf.config`, controls how `DictConfig` objects are re-hydrated into Python values. This mode is a shared configuration across all transformers, ensuring consistent behavior during recursive deserialization.

*   **`DictConfig` Mode:** When set to `DictConfig`, the transformer always returns a raw `omegaconf.DictConfig` object, regardless of whether the original object was backed by a structured dataclass.
*   **`DataClass` Mode:** In `DataClass` mode, the transformer attempts to re-hydrate the `DictConfig` into its original structured dataclass type. This mode requires the dataclass definition to be available in the execution environment.
*   **`Auto` Mode:** The `Auto` mode intelligently determines the appropriate return type. If the serialized `DictConfig` contains information about an underlying structured dataclass, it will attempt to re-hydrate into that dataclass. Otherwise, it defaults to returning a `omegaconf.DictConfig` object.

Using `DataClass` or `Auto` mode with structured configs (dataclasses) is generally recommended for improved type safety and better developer experience, as it allows for static analysis and IDE auto-completion.

### Usage Examples

To use OmegaConf objects in your Flyte tasks, simply type-hint your task inputs and outputs with `omegaconf.DictConfig` or `omegaconf.ListConfig`.

**Example: Passing a `DictConfig` as input and output**

```python
from flytekit import task, workflow
from omegaconf import DictConfig, OmegaConf

@task
def process_config(config: DictConfig) -> DictConfig:
    """
    A task that takes a DictConfig, modifies it, and returns it.
    """
    print(f"Received config: {OmegaConf.to_yaml(config)}")
    config.new_key = "added_value"
    return config

@workflow
def config_workflow(initial_config: DictConfig) -> DictConfig:
    """
    A workflow demonstrating DictConfig passing.
    """
    updated_config = process_config(config=initial_config)
    return updated_config

# Example execution
if __name__ == "__main__":
    my_config = OmegaConf.create({"model": {"name": "resnet", "version": 50}})
    result_config = config_workflow(initial_config=my_config)
    print(f"Workflow output config: {OmegaConf.to_yaml(result_config)}")
    # Expected output will include 'new_key: added_value'
```

**Example: Using `ListConfig` with mixed types**

```python
from flytekit import task, workflow
from omegaconf import ListConfig, OmegaConf

@task
def analyze_list(data_list: ListConfig) -> ListConfig:
    """
    A task that processes a ListConfig with mixed types.
    """
    print(f"Received list: {OmegaConf.to_yaml(data_list)}")
    data_list.append("new_item")
    return data_list

@workflow
def list_workflow(input_list: ListConfig) -> ListConfig:
    """
    A workflow demonstrating ListConfig passing.
    """
    processed_list = analyze_list(data_list=input_list)
    return processed_list

# Example execution
if __name__ == "__main__":
    mixed_list = OmegaConf.create([1, "hello", {"a": 1}, True])
    result_list = list_workflow(input_list=mixed_list)
    print(f"Workflow output list: {OmegaConf.to_yaml(result_list)}")
    # Expected output will include 'new_item'
```

**Example: `DictConfig` with a structured dataclass**

For better type safety and maintainability, define your configurations using Python dataclasses and instantiate them with OmegaConf.

```python
from dataclasses import dataclass
from flytekit import task, workflow
from omegaconf import DictConfig, OmegaConf

@dataclass
class ModelConfig:
    name: str
    version: int
    optimizer: str = "Adam" # Default value

@task
def train_model(config: DictConfig) -> str:
    """
    A task that expects a DictConfig, potentially backed by ModelConfig.
    """
    # If OmegaConfTransformerMode is DataClass or Auto, 'config' will be a ModelConfig instance
    # Otherwise, it will be a DictConfig.
    if OmegaConf.is_config(config) and OmegaConf.get_type(config) == ModelConfig:
        print(f"Received structured config: {config.name} v{config.version} with {config.optimizer}")
        return f"Training {config.name} v{config.version}"
    else:
        print(f"Received generic DictConfig: {OmegaConf.to_yaml(config)}")
        return "Training with generic config"

@workflow
def training_workflow(model_cfg: DictConfig) -> str:
    """
    A workflow demonstrating structured config passing.
    """
    return train_model(config=model_cfg)

# Example execution
if __name__ == "__main__":
    # Create a DictConfig from a dataclass
    my_model_config = OmegaConf.structured(ModelConfig(name="EfficientNet", version=b7))
    result = training_workflow(model_cfg=my_model_config)
    print(f"Workflow result: {result}")
```

### Important Considerations

*   **Type Preservation:** The `DictConfigTransformer` and `ListConfigTransformer` embed Python type information within the serialized `STRUCT` literal. This allows the transformers to correctly re-hydrate complex or heterogeneous OmegaConf objects, including those backed by dataclasses or lists with mixed types.
*   **`omegaconf.MISSING`:** The integration explicitly handles `omegaconf.MISSING` values. When a missing value is encountered in a `ListConfig` or `DictConfig`, it is serialized as a special marker and correctly re-hydrated as `omegaconf.MISSING` on the receiving end.
*   **No Direct LiteralType Introspection:** While the transformers preserve rich type information internally, the Flyte `LiteralType` for `DictConfig` and `ListConfig` is always `SimpleType.STRUCT`. This means you cannot directly inspect the types of nested elements from the Flyte type system itself; the re-hydration logic within the transformers handles this.
*   **Dependencies:** Ensure `omegaconf` is installed in your Flyte task execution environment.
*   **Configuration of `OmegaConfTransformerMode`:** The mechanism for setting the `OmegaConfTransformerMode` (e.g., globally or per task) is typically managed through Flytekit's plugin configuration. Consult the Flytekit documentation for details on configuring plugins.
<!--
key: summary_omegaconf_integration_7275b047-1750-4d4d-ba57-9e9e87dd1657
type: summary_end

-->
<!--
code_unit: flytekitplugins.omegaconf.examples.omegaconf_usage
code_unit_type: class
help_text: ''
key: example_57a0ba36-9000-4763-84d7-b048d9ae5de1
type: example

-->