# Upgrading to 3.x

Please reference the [New Features](http://docs.taffy.io/3.0.0/#New-Features) and [Breaking Changes](http://docs.taffy.io/3.0.0/#Breaking-Changes) sections of the documentation.

## Representation Classes are now called Serializers

`taffy.core.baseRepresentation` is now `taffy.core.baseSerializer`; `taffy.core.nativeJsonRepresentation` is now `taffy.core.nativeJsonSerializer`; etc.

## A single Serializer method may support multiple requested Response Mime Types

Where previously you could only support one mime type per `getAsFoo()` method, you may now supply a comma-delimited list to `taffy:mime` and that method will be used if any of its list values is the requested response mime type: `taffy:mime="application/json,text/json"`

## We now have Input Deserializers

These allow you to accept multiple data formats as input. Just as with Serializers, the default uses ColdFusion's native JSON (de)Serialize functionality. You can extend or replace it as you see fit. [More information here](http://docs.taffy.io/3.0.0/#Custom-Deserializers).

## Caching Hooks

If you're using things like EHCache, you may find benefit in the new [Caching Hooks](http://docs.taffy.io/3.0.0/#Caching-Hooks).

## newRepresentation() has been removed from Application.cfc

When you want to abort a request from inside `onTaffyRequest()`, just use `noData()` or `representationOf()` just as you would from inside a resource.

# Upgrading to 2.x

## applicationStartEvent and requestStartEvent were removed

In all 1.x versions of Taffy, we asked you to use `applicationStartEvent()` instead of `onApplicationStart()` and `requestStartEvent()` instead of `onRequestStart()`. The only benefit to this approach was not having to explain how, where, and when to use `super.onApplicationStart()` and `super.onRequestStart()`. But you're smart people, so let's just rip this band-aid off now, while the ripping is good.

**There is no 100% "always do it this way" guarantee... but there is a rule of thumb.**

There's a good chance you're not doing anything crazy in your Application.cfc; or at least I hope so. For the sake of explanation, let's say that in your old `applicationStartEvent()` method you defined a datasource application variable, and in your old `requestStartEvent()` method you injected some CGI variable into the request scope. (Yes, this example is contrived. Can you do better? :P)

**OLD, BAD, DEPRECATED CODE:**

```cfs
component extends="taffy.core.api" {

    function applicationStartEvent(){
        application.dsn = "myDSN";
    }

    function requestStartEvent(targetPath){
        request.https = CGI.https;
    }

}
```

How do you rewrite this old, bad, deprecated style in the new 2.x hotness format? Like this... (hold on to your butts...)

**NEW HOTNESS:**

```cfs
component extends="taffy.core.api" {

    function onApplicationStart(){
        application.dsn = "myDSN";
        super.onApplicationStart();
    }

    function onRequestStart(targetPath){
        request.https = CGI.https;
        super.onRequestStart(arguments.targetPath);
    }

}
```

Yes folks, it's that easy. Rename the methods, and add `super.[same-method-name]()` at the end.

### YMMV: Your Mileage May Vary

Just because this example shows that you should call `super.[same-method-name]()` at the end of your implementations doesn't mean you can't call it earlier. Just be aware that when you call `super.onApplicationStart()` Taffy inspects `variables.framework` to setup the framework config (so you'll need to setup your 3rd party bean factory _BEFORE_ calling super...). Similarly, when you call `super.onRequestStart(targetPath)` Taffy handles reload and dashboard requests, and a few other odds and ends.

If you're not sure what's right, [ask on the mailing list](https://groups.google.com/forum/#!forum/taffy-users)!

## Deprecated features have been obsoleted

### Request-level Representation Class Overrides

Prior to Taffy 1.1 you could pass an additional argument to the `representationOf()` method in your resource CFC, as well as to the `newRepresentation()` method when in `onTaffyRequest()` that would use a representation class that differed from your default. This feature was deprecated in 1.1 and instead all supported mime types should be implemented by a single representation class. That class is now defined as [variables.framework.representationClass](http://docs.taffy.io/2.2.4/#representationClass)

### registerMimeType and setDefaultMimeType

These methods (of Application.cfc) were deprecated in Taffy 1.1 in favor of [Configuration via Metadata](http://docs.taffy.io/2.2.4/#Configuration-via-Metadata).

### configureTaffy and its child setter methods

These methods were deprecated in Taffy 1.2 in favor of [variables.framework](http://docs.taffy.io/2.2.4/#variables-framework-settings).

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

At the same time, a new variables.framework setting was introduced to define Taffy's behavior [if the dashboard were to be requested while disabled](http://docs.taffy.io/2.2.4/#disabledDashboardRedirect).