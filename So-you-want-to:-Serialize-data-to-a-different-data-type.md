_This example is implemented in the folder `examples/api_anythingtoxml/` (included in the download)._

Taffy has been designed from the start to make it simple to support multiple return formats, and to decouple this serialization of native ColdFusion data types into the desired return format from your "model" (resources). In this example, I'll show how you can use the terrific [AnythingToXML](http://anythingtoxml.riaforge.org) library to enable you to return XML instead of JSON.

## First let's create a representation class

A representation class is what Taffy uses to serialize your data from native CF data types to the desired result. You can write your own representation classes and use them in your API's to enable more formats.

**resources/AnythingToXMLRepresentation.cfc**

We'll put our representation class in the resources subfolder of the api so that Taffy can find and use it.

```cfm
<cfcomponent extends="taffy.core.baseRepresentation">

	<cfset variables.anythingToXml = application.anythingToXml />

	<cffunction
		name="getAsXML"
		output="false"
		taffy:mime="application/xml">
			<cfreturn variables.anythingToXml.toXml(variables.data) />
	</cffunction>

</cfcomponent>
```

If you're following along closely, you noticed that we haven't put AnythingToXML in the Application scope, but we're referencing it there. We'll create it in the next section, so for now just assume it's already there.

>**A quick word on this application variable usage:** AnythingToXML is being saved in Application scope because reinstantiating it for every request would be inefficient. Instead we can reuse the cached instance for every request. When using an external library to assist in serialization, this is the recommended path. 
>
>Technically, yes, this "breaks encapsulation". If you want to jump through a bunch of hoops and inject it into your representation class instance, be my guest. I'm showing it this way to keep the example simple. We'll cover Dependency Injection later! _And hey, rules are made to be broken, right?_

There are a few conventions to follow when creating a representation class:

* Extend the class `taffy.core.baseRepresentation`. This parent class provides a lot of helper methods and useful functionality for representations. It also helps keep your code devoid of boilerplate, and that's a good thing.
* The method that does the serialization should be named `getAsX` where X is the name of the format you're serializing to (XML in this case).
* The function attribute `taffy:mime` describes the Content-Type that will be sent back to the consumer (application/xml in this case).
* The native data passed to your representation will be available as the variable `variables.data`. This is what you should serialize.

## Application.cfc

Now that we have a representation class capable of serializing to the desired format, we need to tell Taffy to use it. Out of the box and unless you change it, Taffy uses the class `taffy.core.nativeJsonRepresentation` to serialize returned data. To use our AnythingToXML class from above, we need to add a few lines to Application.cfc:

```cfm
<cfcomponent extends="taffy.core.api">
	<cfscript>

		this.name = "taffy_xml_example";

		function applicationStartEvent(){
			application.anythingToXml = createObject("component", "anythingtoxml.AnythingToXML").init();
		}

		function configureTaffy(){
			setDefaultRepresentationClass("resources.AnythingToXmlRepresentation");
		}

	</cfscript>
</cfcomponent>
```

Here you can see that we've instantiated the missing variable `application.anythingToXml` from the previous section. Note that this is done inside `applicationStartEvent()` -- you should use this method to do things that you don't want repeated on every request (things you would normally put in `onApplicationStart()`). When you reload the framework, this method is called and variables here will be re-evaluated.

Next, we've added the method `configureTaffy()` and called the framework method `setDefaultRepresentationClass()` inside it, passing in the path to our new custom representation class. ConfigureTaffy is similar to applicationStartEvent, it is not run on every request, but it is called after framework initialization has happened, to give you the opportunity to modify framework defaults. There are other things you can do from configureTaffy, but they are out of scope for this example.

## Changes to your resources

Absolutely nothing. Seriously. That's the _entire_ point of decoupling resources from representations: you can change one without affecting the other!

## I lied, a little bit.

Now that you understand how to create your own custom representation class and tell Taffy to use it, I have a confession to make. If you want to do exactly what I've shown here -- that is, use AnythingToXML and make XML your default and only supported format, I've already done most of the work for you. I only used it as the example for the universal understanding that people seem to have about XML. In actuality, Taffy also comes with the class `taffy.bonus.AnythingToXMLRepresentation` which works exactly as I've coded it above. You don't need to write the representation, you only need to create the application variable `application.anythingToXML` and wire the bonus class into your API like this:

```cfs
function applicationStartEvent(){
	application.anythingToXml = createObject("component", "anythingtoxml.AnythingToXML").init();
}
function configureTaffy(){
	setDefaultRepresentationClass("taffy.bonus.AnythingToXMLRepresentation");
}
```

Can you forgive me?