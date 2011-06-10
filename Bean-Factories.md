In addition to fairly tight integration with ColdSpring, should you choose to use it, Taffy comes with a simple bean factory to manage the file conventions it follows.

TOC:

* Included Bean Factory
* External Bean Factory Integration
 * Using a New Bean Factory Instance
 * Using an Existing Bean Factory Instance
* Supported Bean Factories

## Included Bean Factory

* Collection and Member Resources from the `/resources` folder
* Representation classes stored in the `/resources` folder
* Miscellaneous classes stored in the `/resources` folder, used to resolve dependencies

Resources are identified by the fact that they ultimately extend the `taffy.core.resource` class. Representations are identified by the fact that they ultimately extend the `taffy.core.baseRepresentation` class. Everything else is treated as an asset that can be used to resolve dependencies of Resources, Representations, or other assets.

All Resources (cfc's that extend `taffy.core.resource`) will be treated as **Singletons**: A single instance will be created and cached by the framework for improved efficiency. For this reason, you should be careful to build your Resources to be thread-safe, by using `var` and locks where appropriate. All Representations will be treated as **Transients**: A new instance will be created from scratch each time it is needed, and thrown away after the request is complete.

## External Bean Factory Integration

You have two options when using external bean factories with Taffy. You can use a new factory instance to manage Taffy assets (resources, representations, and miscellaneous assets they depend on); or you can use a factory instance previously loaded by your core application.

### Using a New Bean Factory Instance

To make Taffy use a new external bean factory instance, you just need to instantiate it and pass it to the `setBeanFactory` method. In the case of ColdSpring, you start with a ColdSpring config XML file. You can, if you like, keep your coldspring.xml file in the `/resources` folder. I might. But it doesn't really matter to Taffy where you put it.

To instantiate the bean factory and tell Taffy about it, your **Application.cfc** should look something like this:

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

### Using an Existing Bean Factory Instance

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
	component
		extends="taffy.core.api"
	{
		this.name = 'MyExampleApplication'; //same app name as core app
		
		function configureTaffy()
		{
			setBeanFactory(application.beanFactory);
		}
	}
```

Notice that I did not instantiate the bean factory in the Taffy application, but referenced it there. That is only possible when you use the same Application Name.

## Supported Bean Factories

ColdSpring is probably the most used bean factory that I know of, but that doesn't mean it should be the only one Taffy supports. If you want to use another Bean Factory with Taffy, [let me know](/atuttle/taffy/issues) and I'll be happy to add support for it. Or, in the spirit of open source, you could fork and add the functionality yourself, then send me a pull request. :)