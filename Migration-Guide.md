## Upgrading to 2.x

### `applicationStartEvent` and `requestStartEvent` (were removed)

...

### Deprecated features have been obsoleted

#### Custom Representation Classes

Prior to Taffy 1.1 you could pass an additional argument to the `representationOf()` method in your resource CFC, as well as to the `newRepresentation()` method when in `onTaffyRequest()` that would use a representation class that differed from your default. This feature was deprecated in 1.1 and instead all supported mime types should be implemented by a single representation class. That class is now defined as [variables.framework.representationClass](https://github.com/atuttle/Taffy/wiki/List-of-all-variables.framework-settings#representationclass)

