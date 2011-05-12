In addition to fairly tight integration with ColdSpring, should you choose to use it, Taffy comes with a simple bean factory to manage the file conventions it follows.

* [Included Bean Factory](#included_bean_factory)
* [External Bean Factory Integration](#external_factory)
 * [Using a New Bean Factory Instance](#new_factory_instance)
 * [Using an Existing Bean Factory Instance](#existing_factory_instance)
* [Supported Bean Factories](#supported_factories)

<h2 id="included_bean_factory">Included Bean Factory</h2>

* Collection and Member Resources from the /resources folder
* Representation classes stored in the /resources folder
* Miscellaneous classes stored in the /resources folder, used to resolve dependencies

Resources are identified by the fact that they ultimately extend the `taffy.core.resource` class. Representations are identified by the fact that they ultimately extend the `taffy.core.baseRepresentation` class. Everything else is treated as an asset that can be used to resolve dependencies of Resources, Representations, or other assets.

All Resources (cfc's that extend `taffy.core.resource`) will be treated as **Singletons**: A single instance will be created and cached by the framework for improved efficiency. For this reason, you should be careful to build your Resources to be thread-safe, by using `var` and locks where appropriate. All Representations will be treated as **Transients**: A new instance will be created from scratch each time it is needed, and thrown away after the request is complete.

<h2 id="external_factory">External Bean Factory Integration</h2>

You have two options when using external bean factories with Taffy. You can use a new factory instance to manage Taffy assets (resources, representations, and miscellaneous assets they need); or you can use the factory instance already loaded and being used by your core application.

<h3 id="new_factory_instance">Using a New Bean Factory Instance</h3>

To make Taffy use a new external bean factory instance, you just need to instantiate it and pass it to the `setBeanFactory` method. In the case of ColdSpring, you start with a ColdSpring config XML file. You can, if you like, keep your coldspring.xml file in the `/resources` folder. I might. But it doesn't really matter where you put it.

To create the bean factory and tell Taffy about it, your **Application.cfc** should look something like this:

```cfs
	component extends="taffy.core.api"
	{
		this.name = 'myapi';
		
		function applicationStartEvent(){
			application.beanFactory = createObject("coldspring.beans.DefaultXmlBeanFactory").init();
			application.beanFactory.loadBeansFromXML('/resources/coldspring.xml');
		}
		
		function configureTaffy(){
			setBeanFactory(application.beanFactory);
		}
	}
```

<h3 id="existing_factory_instance">Using an Existing Bean Factory Instance</h3>

It is possible to manage Taffy resources and representations in an external bean factory instance that originates outside your Taffy application context. Typically, this means you have an existing application, using a bean factory such as ColdSpring. To make it work with Taffy, first inform it about your Taffy Resources and Representations (i.e. add them to its configuration file), and then tell Taffy where to find the bean factory.

Normally, your Taffy API would not have access to the bean factory from your core application, because it is its own application. However, we can take advantage of a feature of ColdFusion and share the application context between applications by giving them the same **Application Name**.

Here is an example of the above described scenario.

**Core Application, Application.cfc:**

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

**Taffy API, Application.cfc:**

```cfs
	component
		extends="taffy.core.api"
	{
		this.name = 'MyExampleApplication';
		
		function configureTaffy()
		{
			setBeanFactory(application.beanFactory);
		}
	}
```

Notice that I did not create my bean factory in the Taffy application, but I referenced it there. That is only possible when you use the same Application Name.

<h2 id="supported_factories">Supported Bean Factories</h2>

ColdSpring is probably the most used bean factory that I know of, but that doesn't mean it should be the only one Taffy supports. If you want to use another Bean Factory with Taffy, [let me know](/atuttle/taffy/issues) and I'll be happy to add support for it.