In addition to integrating easily with ColdSpring and (as of Taffy 1.3) DI/1, Taffy comes with its own simple bean factory to manage the file conventions it provides.

TOC:

* How does the included bean factory work?
* External Bean Factories
 * I don't want to share anything with an existing application, I want my API entirely segregated.
 * I want to share my bean factory with my existing application and make use of its cached beans.
 * What bean factories are supported?

## How does the included bean factory work?

The included bean factory manages all of the objects in your `/resources` folder, which will fall into one of three categories:

* Collection and Member Resources from the `/resources` folder
* Representation classes stored in the `/resources` folder
* Miscellaneous classes stored in the `/resources` folder, used to resolve dependencies

Resources are identified by the fact that they ultimately extend the `taffy.core.resource` class. Representations are identified by the fact that they ultimately extend the `taffy.core.baseRepresentation` class. Neither need to be directly inherited -- that is, you can create reusable classes that extend from one of the taffy core classes, and then extend those to create your resources and representations. Everything else is treated as an asset that can be used to resolve dependencies of Resources, or Representations.

All Resources (cfc's that extend `taffy.core.resource`) will be treated as **Singletons**: A single instance will be created and cached by Taffy for improved efficiency. For this reason, you should be careful to build your Resources to be thread-safe, by using `var` and locks where appropriate. All Representations will be treated as **Transients**: A new instance will be created from scratch each time it is needed, and thrown away after the request is complete.

## External Bean Factories

You have two options when using external bean factories with Taffy. You can use a new factory instance to manage Taffy assets (resources, representations, and miscellaneous assets they depend on); or you can use a factory instance previously loaded by your core application.

This means that if you're already using ColdSpring (or another bean factory) to manage your service layer for your existing application, and you just want to add an API to the applicaiton, you can simply re-use the existing ColdSpring instance in Taffy, if you like. Alternately, you can create another separate instance and use that, if you would rather.

In addition to the two ways you can incorporate an external bean factory into your Taffy API, you can use it in one of two ways:

* Have the factory entirely manage the composition of your Taffy resources. All resources and dependencies are configured in your bean factory configuration.
* The external bean factory is only used to resolve the dependencies of your Taffy resources. In this case, you put your Taffy resource CFCs in the `/resources` folder, and for each `setFoo()` method in each resource, if a bean with the same name exists ("foo" in this case), it will be requested from the factory and passed into the setter.

### I don't want to share anything with an existing application, I want my API entirely segregated.

To make Taffy use a **new** external bean factory instance, you just need to instantiate it and set it into `variables.framework`. In the case of ColdSpring, you start with a ColdSpring config XML file. You can, if you like, keep your coldspring.xml file in the `/resources` folder; I might. It doesn't really matter to Taffy where you put it, so do whatever makes sense to you.

To instantiate the bean factory and tell Taffy about it, your **Application.cfc** should look something like this:

```cfs
	component extends="taffy.core.api"
	{
		this.name = 'myapi';
		variables.framework = {};
		
		function applicationStartEvent(){
			application.beanFactory = createObject("coldspring.beans.DefaultXmlBeanFactory").init();
			application.beanFactory.loadBeansFromXML('/resources/coldspring.xml');
			variables.framework.beanFactory = application.beanFactory;
		}
	}
```

### I want to share my bean factory with my existing application and make use of its cached beans.

It is possible to manage Taffy resources and representations in an external bean factory instance that originates outside your Taffy application folder. Typically, this means you have an existing application, using a bean factory such as ColdSpring. To make it work with Taffy, first inform it about your Taffy Resources and Representations (i.e. add them to its configuration file), and then tell Taffy where to find the bean factory.

Normally, your Taffy API would not have access to the bean factory from your core application, because it has its own application context. However, we can take advantage of a feature of ColdFusion and share the application contexts between applications by giving them the same **Application Name**.

Here is an example of the above described scenario.

**Core Application, Application.cfc:** (perhaps: `/Application.cfc`)

```cfs
	component
	{
		this.name = 'MyExampleApplication';
		
		function onApplicationStart()
		{
			application.beanFactory = createObject("coldspring.beans.DefaultXmlBeanFactory").init();
			application.beanFactory.loadBeansFromXML('/myapp/config/coldspring.xml');
		}		
	}
```

**Taffy API, Application.cfc:** (perhaps: `/api/Application.cfc`)

```cfs
	component extends="taffy.core.api"
	{
		this.name = 'MyExampleApplication'; //NOTE: same app name as core app
		variables.framework = {};
		variables.framework.beanFactory = application.beanFactory;
	}
```

Notice that I did not instantiate the bean factory in the Taffy application, but referenced it there. That is only possible when you use the same Application Name.

### What bean factories are supported?

* [ColdSpring](http://www.coldspringframework.org)
* [DI/1](https://github.com/seancorfield/di1) (as of Taffy 1.3)

If you want to use another Bean Factory with Taffy, [let me know](/atuttle/taffy/issues) and I'll be happy to add support for it. Or, in the spirit of open source, you could fork and add the functionality yourself, then send me a pull request. Some alternative bean factories include [Lightwire](http://lightwire.riaforge.org/) and [WireBox](http://wiki.coldbox.org/wiki/WireBox.cfm). :)