# SchemaHandler

This class manages the reading and writing of data schemas. It provides an interface for interacting with different schema formats through reader and writer implementations. The class supports handling remote input/output operations.

## Attributes

- **name**: str
  - The name of the schema handler.

- **object_type**: Type
  - The type of the object this schema handler is associated with.

- **reader**: Type[[SchemaReader](flytekit_types_schema_types_schemareader)]
  - The schema reader associated with this handler.

- **writer**: Type[[SchemaWriter](flytekit_types_schema_types_schemawriter)]
  - The schema writer associated with this handler.

- **handles_remote_io**: bool = False
  - A boolean indicating whether this handler supports remote I/O operations.



