# So you want to...

This page lists a bunch of tasks you might want to complete while coding with Taffy, and explains how you can accomplish each. It was created as part of Taffy 1.1, so directions here may not always work for Taffy 1.0.

**Table of Contents:**

1. Create a dead-simple CRUD (Create, Read, Update, Delete) API for one resource (aka table of data)
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

## Create a dead-simple CRUD (Create, Read, Update, Delete) API for one resource (aka table of data)

_This example is fully implemented in the folder `examples/api/` (for a few resources)._

### Application.cfc and index.cfm

Every API starts with two simple files: `Application.cfc` and `index.cfm`. The latter, `index.cfm` should always be empty. It doesn't _have_ to be empty, but _its contents will be ignored_ so sometimes I'll put in a note for developers that may come behind me so they understand how it works. That just leaves `Application.cfc`.

All that you need in `Application.cfc` for Taffy to work is the following:

```cfm
<cfcomponent extends="taffy.core.api">
	<cfset this.name = "your_api_app_name" />
</cfcomponent>
```

The way that Taffy works is to take over the traditional request event lifecycle. That is, it operates by doing things in the `onApplicationStart()`, `onRequestStart()`, and `onRequest()` methods. This is possible because of the code: `extends="taffy.core.api"`.

If you would like to run some of your own code for these events, Taffy will call a couple of methods that you can optionally define:

* Instead of `onApplicationStart()`, you should define `applicationStartEvent()`.
* Instead of `onRequestStart()`, you should define `requestStartEvent()`.
* You probably shouldn't use `onRequest()`. If you think you need to, you should probably [ask about it on the mailing list](https://groups.google.com/forum/#!forum/taffy-users), and we'll help you figure out if your use-case does need it and how to proceed.

For example, if you need to include a username and password in your `<cfquery>` tags because they aren't stored in your datasource, you can set application variables like this:

```cfm
<cfcomponent extends="taffy.core.api">
	<cfset this.name = "your_api_app_name" />

	<cffunction name="applicationStartEvent">
		<cfset application.dbUser = "username" />
		<cfset application.dbPass = "password" />
	</cffunction>
</cfcomponent>
```

### Next, you need to add a resource.

There are two types of resources: Collections and Members. You can think of Collections like ColdFusion Query objects, and Members like ColdFusion structs. A collection resource represents a collection of member-resource data. The `/students` URI represents the collection of all students, while `/students/12` URI represents a single member of the students collection. It is because of this URI distinction that we separate collections and members into different ColdFusion Components (CFCs).

You can name the CFCs anything you like, but I tend to name mine `thingCollection` and `thingMember` to be consistent and clear. Put them in the `/resources` subfolder of your API.

**resources/studentCollection.cfc**

```cfm
<cfcomponent extends="taffy.core.resource" taffy:uri="/students">

	<cffunction name="get">
		<cfset var local = {} />
		<cfquery name="local.qGetStudents">
			select * from students
		</cfquery>
		<cfreturn representationOf( local.qGetStudents ) />
	</cffunction>

</cfcomponent>
```

**resources/studentMember.cfc**

```cfm
<cfcomponent extends="taffy.core.resource" taffy:uri="/students/{studentId}">

	<cffunction name="get">
		<cfargument name="studentId" />
		<cfset var local = {} />
		<cfquery name="local.qGetStudent">
			select * from students
			where studentId = <cfqueryparam cfsqltype="cf_sql_integer" value="#arguments.studentId#" />
		</cfquery>
		<cfreturn representationOf( local.qGetStudent ) />
	</cffunction>

</cfcomponent>
```

The differences between these two CFCs might be sort of subtle:

**The `taffy:uri` element is different.** The collection resource uses the value `/students` and the member resource uses the value `/students/{studentId}`. These determine which CFC will be used to respond to a given URL. The part in curly braces ("{}") is called a token, and you can kind of think of it as a variable. When the CFC method is called, the value that shows up in the URL in the same position as the token (so 12 in the case of `/students/12`) is passed to the function argument that has the same name as the token ("studentId"). This way, `/students/12` and `/students/761` return two distinct records. If you want it to, a URI can contain as many tokens as you like. This is perfectly valid: `/courses/{dept}/{courseNum}/{sectionNum}/students/{studentId}`.

Aside from the query that is run and the URI differences just described, these components are basically identical.

* Because the method is named "get", it will be called when the request verb is GET. If for some reason you don't want to name your method "get", you can name it anything you like and use the function attribute `taffy:verb="get"` to specify that this function should be called for GET requests. The same conventions apply for all other verbs as well.
* Because no other methods are defined in these components requests using verbs other than GET (like POST, PUT, and DELETE) will automatically be refused with an HTTP status of [405 Method Not Allowed](http://en.wikipedia.org/wiki/List_of_HTTP_status_codes).

**Let's allow student records to be deleted**

To keep the example simple, we're going to ignore security. Depending on your data, chances are good you don't want just anyone deleting it. We'll cover security later.

While you could implement delete for _**an entire collection**_ in a similar manner, it will be a rare, rare API that gives you this sort of freedom. Instead, let's write a custom response to add a little personality to the result:

**in resources/studentCollection.cfc**

```cfm
<cffunction name="delete">
	<cfreturn
		noData()
		.withStatus(405, "Method Not Allowed")
		.withHeaders(
			{"X-FAIL-MESSAGE"="This attempt to delete all students has been added to your permanent record."}
		)
	/>
</cffunction>
```

As previously mentioned, Taffy will automatically deny a DELETE request if you haven't implemented a delete method. But this example does give us an opportunity to introduce a few more bits of the framework without much overhead. Four things are new here: `noData()`, `withStatus()`, `withHeaders()`, and method chaining.

* `noData()` tells Taffy that you want to return an empty response body to the client.
* `withStatus()` sets the HTTP Status Code of the response. In this example we've used 405 to indicate that this method (delete) is not allowed. The second argument is the status text that would accompany the status code. (For example, in "200 OK", "200" is the status code, and "OK" is the status text.)
* `withHeaders()` allows you to return custom headers for the current request. You pass it a structure whose key names will be the header names (usually custom headers should start with an X), and whose values are the header values. Note that the implicit struct notation used above is not compatible with ColdFusion 8; there you must declare and assign values to the structure before passing it to the method.
* Inspired very much by jQuery, Taffy encourages a lot of method chaining. This means simply that you can fit a lot of information, expressively, into one line. This is method chaining: `noData().withStatus().withHeaders()` -- each method in the chain modifys the object returned by the first method in the chain, but then returns that same object, so that the next method in the chain can modify it, and so on, until you stop chaining.

To delete a single student record, you'll want to add a delete method to the student member resource:

**resources/studentMember.cfc**

```cfm
<cffunction name="delete">
	<cfargument name="studentId" />

	<cfset var local = {} />
	<!--- check that the student exists first --->
	<cfquery name="local.qCheckStudentExists">
		select count(studentId) from students
		where studentId = <cfqueryparam cfsqltype="cf_sql_integer" value="#arguments.studentId#" />
	</cfquery>
	<cfif local.qCheckStudentExists.count eq 0>
		<cfreturn noData().withStatus(404, "Not Found") />
	</cfif>

	<!--- student found, delete them --->
	<cfquery name="local.qDeleteStudent">
		delete from students
		where studentId = <cfqueryparam cfsqltype="cf_sql_integer" value="#arguments.studentId#" />
	</cfquery>

	<cfreturn noData() />
</cffunction>
```

In this example you can see that we've added a little bit of input validation. Before performing the delete, we check to see if the record exists, and if not, return 404 Not Found.

### Inserting and Updating works the same way.

You'll write a method that responds for the "POST" (insert) and "PUT" (update) methods, do appropriate input validation, and then run some insert or update sql. Status code 201 is used to indicate a successful insert. The only significant difference is that in some--perhaps most--cases, POST and PUT methods should return the inserted/updated record for confirmation. At a minimum, your post method should set a header to return the created record's ID value (if you're using an identity key) -- I like "X-INSERTED-ID".

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






