This document describes how to use a custom class to serialize your return data. Using a custom representation class will allow you to serialize data to formats other than JSON, the default/included serializer. For example, you could write a custom representation class that allows you to serialize data to XML, WDDX, YAML, RSS, plain text, HTML, or pretty much any other format you can think of; including custom or proprietary formats.

## Requirements

In order to create a custom representation class, you need to know what a representation class is made up of. This is the **baseRepresentation** class included with Taffy, capable of serializing to JSON on CF8 and later:

```cfm
<cfcomponent output="false" hint="a helper class to represent easily serializable data">

	<cfset variables.data = "" />
	<cfset variables.statusCode = 200 />
	<cfset variables.miscHeaders = StructNew() />

	<cffunction name="setData" access="public" output="false" hint="setter for the data to be returned">
		<cfargument name="data" required="true" hint="the simple or complex data that you want to return to the api consumer" />
		<cfset variables.data = arguments.data />
		<cfreturn this />
	</cffunction>

	<cffunction name="noData" access="public" output="false" hint="returns empty representation instance">
		<cfreturn this />
	</cffunction>

	<cffunction name="withStatus" access="public" output="false" hint="used to set the http response code for the response">
		<cfargument name="statusCode" type="numeric" required="true" hint="eg 200" />
		<cfset variables.statusCode = arguments.statusCode />
		<cfreturn this />
	</cffunction>

	<cffunction name="getStatus" access="public" output="false" returnType="numeric">
		<cfreturn variables.statusCode />
	</cffunction>

	<cffunction name="withHeaders" access="public" output="false" hint="used to set custom headers for the response">
		<cfargument name="headerStruct" type="struct" required="true" />
		<cfset variables.miscHeaders = arguments.headerStruct />
		<cfreturn this />
	</cffunction>

	<cffunction name="getHeaders" access="public" output="false" returntype="Struct">
		<cfreturn variables.miscHeaders />
	</cffunction>

</cfcomponent>
```

These are the things that your custom representation class must implement:

* `setData(any data)` - used to tell the class what data it has to serialize. The **data** parameter should accept any data type, and the function should return the **this** object.<br/><br/>
* `noData()` - used to return a (more or less) empty representation object (status information with no result data). Should return the **this** object.<br/><br/>
* `getAsX()` - Used to get the serialized version of the data to be returned to the user. X will correspond with the available formats for your API. If you provide XML, JSON, and YAML mime types as options, your custom representation class should implement all three of the following: getAsXml, getAsJson, and getAsYaml. Note that these _do not_ go in different classes - the one class must be able to serialize the data into all available formats.<br/><br/>
* `withStatus(numeric statusCode)` - used to set the return http status code. Should return the **this** object.<br/><br/>
* `getStatus()` - used by the framework to get the http status code to return to the consumer.<br/><br/>
* `withHeaders(struct headers)` - used to set return http headres. Returns the **this** object.<br/><br/>
* `getHeaders()` - used by the framework to get the custom http headers to add to the response, if any. Returns the headers structure.

It should be pretty obvious that a majority of the above code would be duplicated letter for letter in your custom representation class, so if you like you can subclass the baseRepresentation class and only implement the functions that you want to override or add new serialization functions. For example, here's a custom class that adds XML as a mime type:

```cfm
<cfcomponent output="false" extends="taffy.core.baseRepresentation">

	<cffunction name="getAsXml" access="public" output="false" taffy:mime="application/xml">
		<cfreturn serializeXML(variables.data) />
	</cffunction>

</cfcomponent>
```

... of course, it would be up to you to implement the **serializeXML** function (and I'm a fan of [AnythingToXML](http://anythingtoxml.riaforge.org/) for this); but the important thing to notice here is that none of the other functions need to be duplicated. Since they would be the same, you can just inherit them from the included **baseRepresentation** class.

## Overriding the _default_ representation class

Instead of supplying a custom representation class path as the 2nd argument to the **representationOf** method in every single responder, you can override the global default. To do so, use the **[setDefaultRepresentationClass](/atuttle/Taffy/wiki/Index-of-API-Methods#setDefaultRepresentationClass)** method in conjunction with **[configureTaffy](/atuttle/Taffy/wiki/Index-of-API-Methods#configureTaffy)**.

## A word of caution

Representation classes are treated as **Transient objects** in Taffy, meaning that a new response object is instantiated for every request, which has both positive and negative side-effects.

On the positive side, there is no chance that wires will get crossed and the response to Request A will be sent for Request B because you forgot to var-scope something. On the other hand, it also means that your representation class should be as lightweight as possible to speed instantiation time (and your API's performance may suffer if you don't follow this advice). If you are using a library to do serialization (perhaps you're using [JSONUtil](http://jsonutil.riaforge.org/) for JSON, or [AnythingToXML](http://anythingtoxml.riaforge.org/) for XML) then you should cache the worker object in a persistent scope like Application or Server, because the library does not need to be re-instantiated on every request. Doing so will definitely help the performance of your API.
