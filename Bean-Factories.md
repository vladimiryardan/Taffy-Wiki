In addition to fairly tight integration with ColdSpring, should you choose to use it, Taffy comes with a simple bean factory to manage the file conventions it follows.

#Included Bean Factory

* Collection and Member Resources from the /resources folder
* Representation classes stored in the /resources folder
* Miscellaneous classes stored in the /resources folder, used to resolve dependencies

Resources are identified by the fact that they ultimately extend the `taffy.core.resource` class. Representations are identified by the fact that they ultimately extend the `taffy.core.baseRepresentation` class. Everything else is treated as an asset that can be used to resolve dependencies of Resources, Representations, or other assets.

All Resources (cfc's that extend `taffy.core.resource`) will be treated as **Singletons**: A single instance will be created and cached by the framework for improved efficiency. For this reason, you should be careful to build your Resources to be thread-safe, by using `var` and locks where appropriate. All Representations will be treated as **Transients**: A new instance will be created from scratch each time it is needed, and thrown away after the request is complete.

#ColdSpring Integration

You have two options when using ColdSpring with Taffy. You can use a new instance of ColdSpring to manage Taffy assets (resources, representations, and miscellaneous assets they need); or you can use the instance of ColdSpring already loaded and being used by your core application.

##Using a new instance of ColdSpring

To use a new instance of ColdSpring as the Bean Factory that Taffy makes use of, you need access to the framework, a ColdSpring config XML file, and a few lines of code to tell Taffy about it. You can, if you like, keep your coldspring.xml file in the `/resources` folder. I might. But it doesn't really matter where you put it.

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

##Using an existing instance of ColdSpring

#Other Bean Factories

ColdSpring is probably the most used bean factory that I know of, but that doesn't mean it should be the only one Taffy supports. If you want to use another Bean Factory with Taffy, [let me know](http://fusiongrokker.com/pages/contact-me) and I'll be happy to add support for it.