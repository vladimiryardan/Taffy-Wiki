# So you want to...

This page lists a bunch of tasks you might want to complete while coding with Taffy, and links to subpages explaining how you can accomplish each of them. It was created as part of Taffy 1.1, so directions here may not always work for Taffy 1.0.

**Index:**

1. [Create a dead-simple CRUD (Create, Read, Update, Delete) API for one resource (aka table of data)](https://github.com/atuttle/Taffy/wiki/So-you-want-to:-Create-a-dead-simple-CRUD-API)
1. Serialize your data to a format other than the default of JSON
1. Serialize your return data to multiple formats (json, xml, ...)
1. Require an API key
1. Rate-limit access to your API
1. Use JSONUtil instead of ColdFusion's native JSON serialization
1. Share application variables between your API and your consumer-facing application
1. Use a bean factory (like ColdSpring) to resolve dependencies of your resources (like configuration without breaking encapsulation)
1. Use a bean factory (like ColdSpring) to completely manage resources
1. Use ColdSpring AOP advice for your resources
1. Write your components using ColdFusion 9+ "script component" syntax

## Serialize your data to a format other than the default of JSON

_This example is fully implemented in the folder `examples/api_anythingtoxml/`._

Taffy has been designed from the start to make it simple to support multiple return formats, and to decouple this serialization of native ColdFusion data types to return format from your "model" (resources). In this example, I'll show how you can use the terriffic [AnythingToXML](http://anythingtoxml.riaforge.org) library to enable you to return XML insetad of JSON.

### First let's create a representation class

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

### Application.cfc

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

### Changes to your resources

Absolutely nothing. Seriously. That's the _entire_ point of decoupling resources from representations: you can change one without affecting the other!

### I lied, a little bit.

Now that you understand how to create your own custom representation class and tell Taffy to use it, I have a confession to make. If you want to do exactly what I've shown here -- that is, use AnythingToXML and make XML your default and only supported format, I've already done most of the work for you. I only used it as the example for the universal understanding that people seem to have about XML. In actuality, Taffy also comes with the class `taffy.bonus.AnythingToXMLRepresentation` which works exactly as I've coded it above. You don't need to write the representation, you only need to create the application variable `application.anythingToXML` and wire the bonus class into your API like this:

```cfm
function applicationStartEvent(){
	application.anythingToXml = createObject("component", "anythingtoxml.AnythingToXML").init();
}
function configureTaffy(){
	setDefaultRepresentationClass("taffy.bonus.AnythingToXMLRepresentation");
}
```

Can you forgive me?

## Serialize your return data to multiple formats (json, xml, ...)

_This example is fully implemented in the folder `examples/api_twoFormats/` (for a few resources)._

In order for your API to be able to respond with multiple formats of the same data, all that you need to have is a single representation class capable of serializing to each of the desired formats. Practically, that means that if your API should be capable of responding with both XML and JSON, you need a representation class (see the previous example for more detail on how to create a representation class) with `getAsXML` and `getAsJSON` methods:

```cfm
<cfcomponent extends="taffy.core.baseRepresentation">

	<cfset variables.anythingToXml = application.anythingToXml />

	<cffunction
		name="getAsXML"
		output="false"
		taffy:mime="application/xml">
			<cfreturn variables.anythingToXml.toXml(variables.data) />
	</cffunction>

	<cffunction
		name="getAsJSON"
		output="false"
		taffy:default="true"
		taffy:mime="application/json">
			<cfreturn serializeJson(variables.data) />
	</cffunction>

</cfcomponent>
```

Something a few people have had trouble with was that they thought they had to have different representation classes for each data format. That is not the case. As in this code, your (single) representation class should be capable of serializing the data into all supported formats.

## Require an API key

_This example is fully implemented in the folder `examples/api_requireApiKey/`._

Taffy exposes a few points in the request lifecycle to you via methods. One of them is `onTaffyRequest`, which is called after the request has been parsed to figure out what it is that's being requested. What your `onTaffyRequest` method returns decides how Taffy will continue.

This is the method signature that Taffy calls, in your Application.cfc: `onTaffyRequest(verb, cfc, requestArguments, mimeExt, headers);`

* **verb:** (string) the verb that the consumer used. GET, POST, PUT, DELETE, etc.
* **cfc:** (string) the cfc that would be used to service the request.
* **requestArguments:** (struct) the arguments that will be sent to the resource method.
* **mimeExt:** (string) the requested "extension" of the return format, so if they want json, it's `json` (as opposed to `application/json`).
* **headers:** (struct) the request headers.

If you implement `onTaffyRequest`, you can return one of two ways:

* **Return TRUE** to allow the request to continue as the consumer intended.
* **Return a Representation Object** to abort the request and return whatever your returned representation object.

If you choose not to implement `onTaffyRequest`, the default implementation always returns true.

You can use this to require an API key. Here's one such approach to do that. Put this in your Application.cfc:

```cfm
function onTaffyRequest(verb, cfc, requestArguments, mimeExt){

	if(not structKeyExists(arguments.requestArguments, "apiKey")){
		return newRepresentation().noData().withStatus(401);//unauthorized because they haven't included their API key
	}

	//api key found
	return true;
}
```

This code checks for a request argument named "apiKey" (i.e. a `GET` request for `/foo?apiKey=abc123`), and if it's not found, returns a blank response body with status code 401. The `newRepresentation` method creates a new instance of your representation class, then you can either use `noData()` or if you want to have a response body, use `setData()` to pass in the data to send back to the consumer. Aside from the `newRepresentation()` and `setData()` methods, this should feel very similar to returning data in your resources. That's essentially what you're doing; just short-circuiting the request.

## Rate-limit access to your API



## Use JSONUtil instead of ColdFusion's native JSON serialization



## Share application variables between your API and your consumer-facing application



## Use a bean factory (like ColdSpring) to resolve dependencies of your resources (like configuration without breaking encapsulation)



## Use a bean factory (like ColdSpring) to completely manage resources



## Use ColdSpring AOP advice for your resources



## Write your components using ColdFusion 9+ "script component" syntax
