_This example is implemented in the folder `examples/api_twoFactories/` (included in the download)._

Taffy comes with a lightweight dependency-injection ("DI") class, simply referred to as its "factory". The factory creates one instance of every cfc ("bean") in your `/resources` folder, and then attempts to resolve dependencies of each of those beans with each other. Dependencies are defined by having a setter function; so if one bean has a `setConfig` method, and a bean named "config" exists (config.cfc), then that bean will be passed to the setter. Learn more about how this works in [Use Taffy's built-in Dependency Injection to resolve dependencies of your resources](https://github.com/atuttle/Taffy/wiki/So-you-want-to:-Use-Taffy's-built-in-Dependency-Injection-to-resolve-dependencies-of-your-resources).

In some cases, you may already be using an external DI framework ("bean factory"), like ColdSpring, and want to use beans already defined and available in that bean factory from your Taffy resources. If that's your use-case, you're in the right place.

### Sharing Application Context

Chances are pretty good that if you're going down this path, you want to share Application Contexts (essentially, share the same set of Application variables), to reduce the amount of memory necessary to run both the _other_ application and your Taffy API. That is described in detail in [Share application variables between your API and your consumer-facing application](https://github.com/atuttle/Taffy/wiki/So-you-want-to:-Share-application-variables-between-your-API-and-your-consumer-facing-application).

Alternatively, you may want to use the same ColdSpring configuration but create a new bean factory instance. There are pro's and con's to each approach, but for simplicity's sake and since the former is already well described in the "sharing application variables" link, this example will assume you're using an external bean factory config, but creating a new instance.

### Create Your Bean Factory

> **Note:** These examples use ColdSpring, but as of Taffy 1.3, DI/1 is also supported.

In `Application.cfc` ~> `applicationStartEvent()` create your bean factory instance, and set it into `variables.framework`:

```cfs
function applicationStartEvent() {
	var beanFactory = createObject("component", "coldspring.beans.DefaultXMLBeanFactory");
	beanFactory.loadBeans('config/coldspring.xml');

	//set bean factory into Taffy
	variables.framework.beanFactory = beanfactory;
}
```

If you set a bean factory like this, **and** do not put anything into the `/resources` folder, then Taffy will use the external factory to get your resources, and will assume that all dependency injection has already been taken care of by that framework. Learn more about this in [Use an external bean factory like ColdSpring to completely manage resources](https://github.com/atuttle/Taffy/wiki/So-you-want-to:-Use-an-external-bean-factory-like-ColdSpring-to-completely-manage-resources).

In this example, however, we're going to put a resource CFC into the `/resources` folder, with a dependency defined, so that Taffy will inject the depended-on bean from ColdSpring.

### Use a setter to have the dependency injected at runtime

In `/resources/artfartCollection.cfc`:

```cfm
	<cfcomponent extends="taffy.core.resource" taffy_uri="/artfarts">

		<cffunction name="get" access="public" output="false">
			<cfreturn representationOf(variables.fakeData.getData()) />
		</cffunction>

		<!--- this will be called by the bean factory's autowire functionality --->
		<cffunction name="setFakeData" access="public" output="false" returnType="void">
			<cfargument name="fakeDataObj" type="any" required="true" hint="Shared FakeData object" />
			<cfset variables.fakeData = arguments.fakeDataObj />
		</cffunction>

	</cfcomponent>
```

Here we have a method named `setFakeData`. Because it begins with the word "set", Taffy asks your external bean factory if it has a bean by the name of "fakedata". If it does, then the bean is requested, and Taffy passes it to the setter.

When requests are made and the resource is used to handle them, the dependency has already been set and you may reference it; as the above example does in its `get` method: `return representationOf(variables.fakeData.getData())`.

### Alternative to setters: Properties

Instead of writing custom setters for everything you want injected, you can simply add properties to your resource CFCs:

```cfs
component extends="taffy.core.resource" taffy_uri="/foo" {

	property name="fakeData";

	public function get() output="false" {
		return representationOf( this.fakeData.getData() );
	}
}
```

The caveat to this approach is that properties live in the `this` scope; whereas when writing your own setters you could choose to set the value into either `this` or `variables`.

### About Caching

Resource CFC's are stored in memory, so this operation only happens on startup (`onApplicationStart`) or on API reload (`index.cfm?reload=true` -- The key and password can be changed, these are the defaults). If you make a change to a dependency or a Taffy resource, it will not take effect until the next reload.

The load order is:

1. Your values from `variables.framework` are inspected and overwrite any defaults that your values would conflict with.
1. `configureTaffy()` is called (in `Application.cfc`)
1. If `/resources` convention folder exists and contains CFC's, Taffy Factory is created and populated.
  * If an external bean factory has been specified, dependencies are further-resolved (after `/resources` resolutions) using external bean factory.
1. If external bean factory has been set (and nothing in `/resources`) then resource classes are loaded from external bean factory under the assumption that their dependencies are already resolved.
1. If the `/resources` folder doesn't exist and an external bean factory hasn't been set, a getting started message is shown with links to various wiki pages.