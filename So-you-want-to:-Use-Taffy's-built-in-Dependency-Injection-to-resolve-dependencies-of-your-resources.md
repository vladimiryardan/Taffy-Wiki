Taffy comes with a lightweight dependency-injection class, simply referred to as its "factory". The default configuration of Taffy uses this factory to enable the use of the `/resources` folder convention.

### `/resources` Folder Convention

When you create a Taffy API, you have two required files: `Application.cfc` and `index.cfm`; and the option of using the `/resources` folder convention. If you create a folder in the root of your API folder (does not necessarily need to be in the web-root of the domain) named "resources", Taffy looks for Taffy Resource CFC's in it; as well as any dependencies that the resources may have.

**A Resource CFC is simply one which extends "taffy.core.resource" and has a "taffy:uri" (or in the case of script-components, "taffy_uri") attribute defining the URI pattern to which the resource will respond.**

### Dependency Resolution

Dependencies in Taffy Resources are defined by the existence of a setter method. If you want Taffy to inject an instance of `/resources/Config.cfc`, simply create a setter method for it like so:

```cfs
	function setConfig(configObj){
		variables.config = arguments.configObj;
	}
```

The argument name does not matter, and what you choose to do with the CFC instance passed to it is up to you; but the name is important. It must begin with "set", and whatever comes after "set" will be the name of the CFC (sans file extension). So in this case, "setConfig" looks for "config.cfc".

All resources and other classes in the resources folder are treated as singletons for the purpose of dependency injection. I hope this doesn't confuse you, but you can (optionally) choose to store a [Custom Representation Class](https://github.com/atuttle/Taffy/wiki/So-you-want-to:-Serialize-data-to-a-different-data-type) in the resources folder, and as long as you're not auto-wiring it into a resource CFC, it will still be treated as a transient object for the purposes of responding to requests. (If you found that confusing, just ignore it. Everything in resources is a singleton.)

Taffy's Factory will **not** look anywhere other than the resources folder, and as of this time of writing, does not do a recursive search (though it is being considered for a future version). There is no configuration for the factory -- it assumes and requires the use of the `/resources` folder.