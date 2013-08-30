# Upgrading to 2.x

## applicationStartEvent and requestStartEvent were removed

...

## Deprecated features have been obsoleted

### Request-level Representation Class Overrides

Prior to Taffy 1.1 you could pass an additional argument to the `representationOf()` method in your resource CFC, as well as to the `newRepresentation()` method when in `onTaffyRequest()` that would use a representation class that differed from your default. This feature was deprecated in 1.1 and instead all supported mime types should be implemented by a single representation class. That class is now defined as [variables.framework.representationClass](https://github.com/atuttle/Taffy/wiki/List-of-all-variables.framework-settings#representationclass)

### registerMimeType and setDefaultMimeType

These methods (of Application.cfc) were deprecated in Taffy 1.1 in favor of [Configuration via Metadata](https://github.com/atuttle/Taffy/wiki/Configuration-via-Metadata).

### configureTaffy and its child setter methods

These methods were deprecated in Taffy 1.2 in favor of [variables.framework](https://github.com/atuttle/Taffy/wiki/List-of-all-variables.framework-settings).

* configureTaffy
* setGlobalHeaders
* getGlobalHeaders
* enableCrossDomainAccess
* setDefaultRepresentationClass
* registerExtensionRepresentation
* setReloadKey
* setReloadPassword
* enableDashboard
* setDashboardKey
* setDebugKey
* setDefaultMime
* setUnhandledPaths
* setBeanFactory
* getBeanFactory

### ?dashboard

In Taffy 1.3 we deprecated using `?dashboard` as in: http://api.example.com/?dashboard in favor of displaying the dashboard when no resource is specified; e.g. http://api.example.com/ or http://example.com/api/v1/ or http://example.com/api/v1/index.cfm

At the same time, a new variables.framework setting was introduced to define Taffy's behavior [if the dashboard were to be requested while disabled](https://github.com/atuttle/Taffy/wiki/List-of-all-variables.framework-settings#disableddashboardredirect).