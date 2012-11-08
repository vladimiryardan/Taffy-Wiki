This page describes how to use a custom class to serialize your return data. Using a custom representation class will allow you to serialize data to formats other than JSON, the default/included serializer. For example, you could write a custom representation class that allows you to serialize data to XML, WDDX, YAML, RSS, plain text, HTML, or pretty much any other format you can think of; including custom, binary, or proprietary formats.

# Writing a custom representation class

To support non-default return formats, you need to create your own representation class. For example, let's say you wanted to support a YML return format. Create a new component (either in your `/resources` folder or in your external bean factory), with the following code:

```cfm
<cfcomponent output="false" extends="taffy.core.baseRepresentation">

	<cffunction name="getAsYml" access="public" output="false" taffy:mime="application/yml">
		<cfreturn serializeYml(variables.data) />
	</cffunction>

</cfcomponent>
```

Implementing a `serializeYml()` method is left as an exercise for the user \(hint: look for an existing library, like [SnakeYaml](http://code.google.com/p/snakeyaml/)\).

## Supporting Multiple return formats

Suppose your api needs to be able to return both JSON and YML representations of your data. You would write your representation class like this:

```cfm
<cfcomponent output="false" extends="taffy.core.baseRepresentation">

	<cffunction name="getAsYml" access="public" output="false" taffy:mime="application/yml">
		<cfreturn serializeYml(variables.data) />
	</cffunction>

	<cffunction name="getAsJson" access="public" output="false" taffy:mime="application/json">
		<cfreturn serializeJson(variables.data) />
	</cffunction>

</cfcomponent>
```

Note that both methods go into the same representation class. Your representation class should be capable of serializing to each return format you want to support.

## Overriding the default representation class

You tell Taffy to use your representation class by supplying either its Bean Name or CFC Dot Notation Path in the setting `variables.framework.representationClass`. See the [[List of all variables.framework settings]] for more information.

## A word of caution

Representation classes are treated as **Transient objects** in Taffy, meaning that a new response object is instantiated for every request, which has both positive and negative side-effects.

On the positive side, there is no chance that wires will get crossed and the response to **Request A** will be sent for **Request B** because you forgot to var-scope something. On the other hand, it also means that your representation class should be as lightweight as possible to speed instantiation time (and your API's performance may suffer if you don't follow this advice). If you are using a library to do serialization (perhaps you're using [JSONUtil](http://jsonutil.riaforge.org/) for JSON, or [AnythingToXML](http://anythingtoxml.riaforge.org/) for XML) then you should cache the worker object in a persistent scope like Application or Server, because the library does not need to be re-instantiated on every request. Doing so will definitely help the performance of your API.

## Getting all advanced up in here (requirements)

These are the things that your custom representation class must implement in order to interact with Taffy, and why:

* `setData(any data)` - used to tell the class what data it has to serialize. The **data** parameter should accept any data type, and the function should return the **this** object to allow for method chaining.<br/><br/>
* `noData()` - used to return a (more or less) empty representation object (status information with no result data). Should return the **this** object to allow for method chaining.<br/><br/>
* `getAsX()` - Used to get the serialized version of the data to be returned to the user. X will correspond with the available formats for your API. If you provide XML, JSON, and YAML mime types as options, your custom representation class should implement all three of the following: getAsXml, getAsJson, and getAsYaml. Note that these _do not_ go into different classes - the one class must be able to serialize the data into all available formats.<br/><br/>
* `withStatus(numeric statusCode)` - used to set the return http status code. Should return the **this** object to allow for method chaining.<br/><br/>
* `getStatus()` - used by the framework to get the http status code to return to the consumer.<br/><br/>
* `withHeaders(struct headers)` - used to set additional return http headers. Should return the **this** object to allow for method chaining.<br/><br/>
* `getHeaders()` - used by the framework to get the custom http headers to add to the response, if any. Returns the headers structure from `withHeaders()`.

You can inherit most of this functionality by extending `taffy.core.baseRepresentation`. This class has all of the helper methods (eg. setData, noData, withStatus, withHeaders, etc), but no serialization methods (eg. getAsJson).

For example, if you wanted to recreate the included class `taffy.core.nativeJsonRepresentation`, you could create your own representation class with the following code:

```cfm
<cfcomponent output="false" extends="taffy.core.baseRepresentation">

	<cffunction name="getAsJson" access="public" output="false" taffy:mime="application/json" taffy:default="true">
		<cfreturn serializeJson(variables.data) />
	</cffunction>

</cfcomponent>
```
