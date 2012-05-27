_This example is implemented in the folder `examples/ParentApplication/` (included in the download)._

One of the core features for Taffy is that it can be "bolted on" to an existing application -- if your primary application framework doesn't support REST or if you just happen to like Taffy, it can coexist with anything. A Taffy API can exist in the web root, or in a sub-folder. It doesn't make a difference; and this is a big part of why Taffy can be bolted on to any existing application.

The benefit of this approach is that you can share variables between two different applications. This is possible because ColdFusion Application Contexts are specified using the application name (`this.name` in Application.cfc). If two Application CFC's have the same value for `this.name`, then they share the same set of application variables.

You can use this to your advantage, for example, by creating a Bean Factory instance (ColdSpring, etc) and having both your public facing application and your api use it from both applications.

Let's start by looking at the "parent" application -- the non-API application.

## 1 problem; 2 common solutions

The only file we need to concern ourselves with from the parent application is `Application.cfc`; and the only part of this file that we need to concern ourselves with is `onApplicationStart` (or its functional equivalent -- some frameworks, like FW/1, have you use another method name for the same purpose).

When sharing an application context, as we're doing in this case, there is a common problem that needs to be addressed: Application initialization. If your parent application sets up the bean factory and the api uses it, but both applications are stopped (e.g. you just restarted the server) and the API is used before the parent application does its initialization, then exceptions will be thrown because variables that the api expects to exist haven't yet been initialized. The same holds true in reverse: if your API does the initialization and your parent application is requested first, it will throw the same exceptions.

There are two popular solutions to this problem:

1. Use a Mixin
1. Self-responsibility

### Using Mixins for application initialization

If you're not familiar with mixins, they are simply an external file that is included in multiple locations for the purpose of keeping your code [DRY][1].

With the Mixin pattern, we simply create an external `.cfm` file that both the API and the Parent application include that sets up all application variables that either would ever need for initialization. Let's assume that this is just the bean factory:

**mixin/appInit.cfm**:
```cfm
<cfset application.beanFactory = createObject("component", "coldspring.beans.DefaultXMLBeanFactory") />
<cfset application.beanFactory.loadBeans('/path/to/coldspring.xml') />
```

And simply include it from your `onApplicationStart` method (or functional equivalent) in the parent application:

```cfm
<cffunction name="onApplicationStart" returnType="boolean" output="false">
	<cfinclude template="mixin/appInit.cfm" />
	<cfreturn true />
</cffunction>
```

As you can imagine, it's just as easy in your api. But instead of `onApplicationStart`, in Taffy you use `applicationStartEvent`:

```cfs
//do your onApplicationStart stuff here
function applicationStartEvent(){
	include "../mixin/appInit.cfm";
}
```

Using this approach, it doesn't matter if the first request is for your api or your application, the same application variables will be initialized.

### Self-responsibility for application initialization

The self-responsibility approach is simply that each application -- the parent application and the api -- knows exactly what application variables it will need, checks for each individually, and creates them if they aren't found.

**Parent application's Application.cfc:**
```cfm
<cffunction name="onApplicationStart" returnType="boolean" output="false">
	<cfif not structKeyExists(application, "beanFactory")>
		<cfset application.beanFactory = createObject("component", "coldspring.beans.DefaultXMLBeanFactory") />
		<cfset application.beanFactory.loadBeans('/path/to/coldspring.xml') />
	</cfif>
	<cfreturn true />
</cffunction>
```

**API's Application.cfc:**

```cfs
//do your onApplicationStart stuff here
function applicationStartEvent(){
	if (!structKeyExists(application, "beanFactory")){
		application.beanFactory = createObject("component", "coldspring.beans.DefaultXMLBeanFactory");
		application.beanFactory.loadBeans('/path/to/coldspring.xml');
	}
}
```

In this example, each application is responsible for creating the variables it will need.

## Which pattern should I choose?

The benefit of the mixin pattern is that you only have to maintain the initialization code in one place. The drawback is the additional file dependency for both applications.

The benefit of the self-responsibility pattern is that the code is portable and self-contained. The drawback is its repetetive nature.

There is no "correct" answer, but in general my preference is for mixins.

[1]:http://en.wikipedia.org/wiki/Don%27t_repeat_yourself
