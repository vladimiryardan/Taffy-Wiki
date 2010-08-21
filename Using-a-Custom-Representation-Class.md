bq. *Warning:* Documentation Currently Outdated<br/><br/>As this product is pre- version 1.0, I am still making breaking changes and not going out of my way to preserve backward compatibility as I come up with better ideas for the ways things can work. The documentation is currently a little bit outdated, but will be updated as part of the 1.0 release.<br/><br/>While the documentation is out of date, the example applications will show working code of varying kinds; as these are my test cases, and I don't develop the core without updating the tests.<br/><br/>_Thanks for your patience as I improve Taffy!_

This document describes how to use a custom class to serialize your return data. Using a custom representation class will allow you to serialize data to formats other than JSON, the default/included serializer. For example, you could write a custom representation class that allows you to serialize data to XML, YAML, RSS, plain text, HTML, or pretty much any other format you can think of; including custom or proprietary formats.

h2. Requirements

In order to create a custom representation class, you need to know what a representation class is made up of. This is the *genericRepresentation* class included with Taffy, capable of serializing to JSON on CF8 and later:

<pre class="prettyprint"><cfcomponent output="false">

	<cfset variables.data = "" />
	<cfset variables.statusCode = 200 />

	<cffunction name="setData" access="public" output="false" returnType="taffy.core.genericRepresentation">
		<cfargument name="data" required="true"/>
		<cfset variables.data = arguments.data />
		<cfreturn this />
	</cffunction>

	<cffunction name="noData" access="public" output="false" returntype="taffy.core.genericRepresentation">
		<cfreturn this />
	</cffunction>

	<cffunction name="getAsJson" access="public" output="false" returntype="String">
		<cfreturn serializeJSON(variables.data) />
	</cffunction>

	<cffunction name="withStatus" access="public" output="false" returntype="taffy.core.genericRepresentation">
		<cfargument name="statusCode" type="numeric" required="true" />
		<cfset variables.statusCode = arguments.statusCode />
		<cfreturn this />
	</cffunction>

	<cffunction name="getStatus" access="public" output="false" returnType="numeric">
		<cfreturn variables.statusCode />
	</cffunction>

</cfcomponent></pre>

These are the things that your custom representation class has to implement:

* @setData(any data)@ - used to tell the class what data it has to serialize. The *data* parameter should accept any data type, and the function should return the *this* object.<br/><br/>
* @noData()@ - used to return a (more or less) empty representation object (status information with no result data). Should return the *this* object.<br/><br/>
* @getAsX()@ - Used to get the serialized version of the data to be returned to the user. X will correspond with the available formats for your API. If you provide XML, JSON, and YAML mime types as options, your custom representation class should implement all three of the following: getAsXml, getAsJson, and getAsYaml. Note that these _do not_ go in different classes - the one class must be able to serialize the data into all available formats.<br/><br/>
* @withStatus(numeric statusCode)@ - used to set the return http status code. Should return the *this* object.<br/><br/>
* @getStatus()@ - used by the framework to get the http status code to return to the consumer.

It should be pretty obvious that a majority of the above code would be duplicated in your custom representation class, so if you like you can subclass the genericRepresentation class and only implement the functions that you want to override or add new serialization functions. For example, here's a custom class that adds XML as a mime type:

<pre class="prettyprint"><cfcomponent output="false" extends="taffy.core.genericRepresentation">

	<cffunction name="getAsXml" access="public" output="false" returntype="String">
		<cfreturn serializeXML(variables.data) />
	</cffunction>

</cfcomponent></pre>

... of course, you need to implement the *serializeXML* function (and I'm a fan of "AnythingToXML":http://anythingtoxml.riaforge.org/ for this); but the important thing to notice here is that none of the other functions need to be duplicated. Since they would be the same, you can just inherit them from the included *genericRepresentation* class.

h2. A word of caution

Representation classes are treated as Transient objects in Taffy, meaning that a new response object is instantiated for every request, which has both positive and negative side-effects.

On the positive side, there is no chance that wires will get crossed and the response to request A will be sent for request B because you forgot to var-scope something. On the other hand, it also means that your representation class should be as lightweight as possible to speed instantiation time (and your API will be slow if you don't follow this advice). If you are using a library to do serialization (perhaps you're using "JSONUtil":http://jsonutil.riaforge.org/ for JSON, or "AnythingToXML":http://anythingtoxml.riaforge.org/ for XML) then you should cache the worker object in a persistent scope like Application or Server, because the library does not need to be re-instantiated on every request. Doing so will definitely help the performance of your API.

<script type="text/javascript" src="http://fusiongrokker.com/assets/plugins/Prettify/assets/prettify.js"></script> 
<script type="text/javascript">function addLoadEvent(func) {var oldonload = window.onload; if (typeof window.onload != 'function') { window.onload = func; } else { window.onload = function() { if (oldonload) { oldonload(); } func(); } } } addLoadEvent(function() { prettyPrint(); });</script>
<link rel='stylesheet' href='http://fusiongrokker.com/assets/plugins/Prettify/assets/prettify.css' type='text/css' media='screen' /> 