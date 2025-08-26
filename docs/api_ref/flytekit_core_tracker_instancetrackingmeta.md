# InstanceTrackingMeta

This metaclass enables tracking of class instances to their originating module. It identifies the module where an instance is created, facilitating the association of instances with their source code. Key features include determining the module name and file, even when instances are created within the __main__ module.

## Constructors
```def InstanceTrackingMeta()
```
-  This is a metaclass, so __init__ is not directly called. Instead, the __call__ method is invoked when an instance of a class using this metaclass is created.



