_This example is implemented in the folder `examples/api_twoFactories/` (included in the download)._

Taffy comes with a lightweight dependency-injection class, simply referred to as its "factory". The factory creates one instance of every cfc ("bean") in your `/resources` folder, and then attempts to resolve dependencies of each of those beans with each other. Dependencies are defined by having a setter function; so if one bean has a `setConfig` method, and a bean named "config" exists (config.cfc), then that bean will be passed to the setter. Learn more about how this works in [Use Taffy's built-in Dependency Injection to resolve dependencies of your resources]().

In some cases, you may already be using an external DI framework ("bean factory"), like ColdSpring, and want to use beans already defined and available in that bean factory from your Taffy resources. If that's your use-case, you're in the right place.

### Sharing Application Context

Chances are pretty good that if you're going down this path, you want to share Application Contexts (essentially, share the same set of Application variables), to reduce the amount of memory necessary to run both the _other_ application and your Taffy API. That is described in detail in [Share application variables between your API and your consumer-facing application]().

Alternatively, you may want to use the same ColdSpring configuration but create a new bean factory instance. There are pro's and con's to each approach, but for simplicity's sake and since the former is already well described in the "sharing application variables" link, this example will assume you're using an external bean factory config, but creating a new instance.

### Create Your Bean Factory

In `Application.cfc` create your bean factory instance, and pass it to Taffy's `setBeanFactory()` method:

```cfs
	var beanFactory = createObject("component", "coldspring.beans.DefaultXMLBeanFactory");
	beanFactory.loadBeans('config/coldspring.xml');

	//set bean factory into Taffy
	setBeanFactory(beanfactory);
```

If you set a bean factory like this, **and** do not put anything into the `/resources` folder, then Taffy will use the external factory to get your resources, and will assume that all dependency injection has already been taken care of by that framework. Learn more about this in [Use an external bean factory (like ColdSpring) to completely manage resources]().

In this example, however, we're going to put a resource CFC into the `/resources` folder, with a dependency defined, so that Taffy will inject the depended-on bean from ColdSpring.

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

### About Caching

Resource CFC's are stored in memory, so this operation only happens on startup (`onApplicationStart`) or on API reload (`index.cfm?reload=true` -- The key and password can be changed, these are the defaults). If you make a change to a dependency or a Taffy resource, it will not take effect until the next reload.

The load order is:

1. `configureTaffy()` is called (in `Application.cfc`), allowing you to set framework settings and supply an external bean factory if desired.
1. If `/resources` convention folder exists and contains CFC's, Taffy Factory is created and populated.
  * If an external bean factory has been specified, dependencies are further-resolved (after `/resources` resolutions) using external bean factory.
1. If external bean factory has been set (and nothing in `/resources`) then resources classes are loaded from external bean factory under the assumption that their dependencies are already resolved.
1. If the `/resources` folder doesn't exist and an external bean factory hasn't been set, an exception is thrown. ("You must either set an external bean factory or use the internal factory by creating a \`/resources\` folder.")