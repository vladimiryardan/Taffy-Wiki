In order to be successful in this endeavor, we're going to assume a few things:

* That you already have a working knowledge of ColdSpring (or your Bean Factory of choice);
* That it _is ColdSpring_, or at least implements [the same interface for bean discovery][1];
* That you have a working knowledge of Taffy Resource development (see: [Getting Started][2]);
* And that you just want to learn how to mash the two together into something beautiful and terse.

For the sake of simplicity, I'll simply refer to all Dependency Injection frameworks as ColdSpring in this document, but as long as they meet our [minimum bean factory requirements][1] you should be able to substitute in whatever factory you like. (Note: Only tested with ColdSpring. If there's a bug when using your favorite bean factory, [let us know!](https://github.com/atuttle/taffy/issues))

## Simple Example

>For the sake of this simple example, let's assume that our API exists in its own bubble: We're using ColdSpring to do dependency injection, but the only code we're concerned with is our Taffy API -- no parent applications or other outside forces to worry about.

_This example is implemented in the folder `examples/api_coldspring/` (included in the download)._

Instead of using Taffy's built in bean factory, we'll create some Taffy resources, define them in our ColdSpring config, and wire in their dependencies. Then Taffy will simply cache references to the Taffy Resource beans contained within ColdSpring.

Let's start with our Taffy Resource:

```cfs
component extends="taffy.core.resource" taffy_uri="/photos" {

	public function setPhotoService(photoService){
		variables.photoService = arguments.photoService;
	}

	public function get() {
		return representationOf(variables.photoService.getPhotos()).withStatus(200);
	}

}
```

While this example uses the "setter" method and, as we will see, ColdSpring's autowire by name functionality, you can just as easily use a the constructor argument method. Any composition method that ColdSpring supports for configuring your objects is allowed.

This would be a good place to look at our coldspring.xml, which tells ColdSpring how to wire the objects together:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans default-autowire="byName">

	<bean id="photoService" class="taffy.examples.shared.fakeData" />
	<bean id="photosCollection" class="taffy.examples.api_coldspring.stuff.photosCollection" />

</beans>
```

It doesn't matter what's in the `photoService` service. All that you need to know is that when Taffy requests the `photosCollection` class, ColdSoring will pre-compose it before giving it to Taffy.

Lastly, of course, we need to run ColdSpring in our Taffy API and tell Taffy about it. Here's Application.cfc:

```cfs
component extends="taffy.core.api" {

	function applicationStartEvent() {
		application.beanFactory = createObject( 'component', 'coldspring.beans.DefaultXMLBeanFactory' );
		application.beanFactory.loadBeans( '/taffy/examples/api_coldspring/config/coldspring.xml' );
	}

	function configureTaffy(){
		setBeanFactory( application.beanfactory );
	}

}
```

### How does this differ from vanilla Taffy?

Just to be explicit, here's what's different:

* You do _not_ put a `/resources` folder inside your api.
* In fact, your resource CFC's don't have to exist inside the API folder at all, if you don't want them to...
  * Your entire API folder could consist of nothing but: **Application.cfc**, **index.cfm**, and **coldspring.xml**.
* You'll probably notice that your resources become extremely terse; simply putting a REST wrapper on your service CFCs.

## Up the difficulty: Let's Integrate!

Once you've got the above example figured out, it's really quite easy to take the next step.

For the sake of a more complicated example, let's say you've already got your application written -- including a service layer for getting at your data, using ColdSpring -- and you're adding on an API after the fact. You're going to use [the "same application name" method][3] to share that ColdSpring instance between the parent and child (API) applications, and ... that's it. So, _profit_, I guess.

### Bonus Achievement: Use AOP to do permissions checks and other neat stuff

For now, this is an exercise left to the reader. Having trouble? Head over to [the mailing list!][4]

[1]: https://github.com/atuttle/Taffy/wiki/Taffy-minimum-requirements-for-3rd-party-Dependency-Injection-frameworks
[2]: https://github.com/atuttle/Taffy/wiki/Getting-Started
[3]: https://github.com/atuttle/Taffy/wiki/So-you-want-to:-Share-application-variables-with-your-consumer-facing-application
[4]: http://groups.google.com/group/taffy-users
