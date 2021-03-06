
Metadata is used to apply configuration in a concise and elegant manor, where necessity and possibility intersect.

<h2 id="Resource_Metadata">In Resources</h2>

Resources are CFCs that interpret a request and provide or update the requested data.

<h3 id="taffy_uri">taffy:uri</h3>

The **taffy:uri** property applies to the `<cfcomponent>` tag or the `component{}` definition. It configures which URIs that the CFC will respond to. There is no limit to the number of tokens that can be used in a single URI, as long as they are not consecutive. Examples:

* CFML:
```cfs
	<cfcomponent taffy:uri="/artist/{artistId}">
	</cfcomponent>
```

* CFScript:
```cfs
	component 
	taffy:uri="/artist/{artistId}"
	{
	}
```
&nbsp;
The following is invalid, because tokens are not separated at all:
```cfs
	taffy:uri="/artist/{artistId}{artistName}"
```
&nbsp;

####Tokens

Tokens in URIs define the parts of the URI that are dynamic. They are identified by curly-brackets: `{tokenName}` and are passed **by name** to all functions that handle the request. The `{foo}` token's value will be passed to the `foo` argument of the associated method.

<h3 id="taffy_verb">taffy:verb</h3>

By convention, resources will automatically map the 4 primary HTTP REST verbs -- GET, PUT, POST, DELETE -- to methods with the same name. For extended verbs -- OPTIONS, HEAD -- or if you want to map one of the primary verbs to a method with a different name, you can use the `taffy:verb` metadata property on the method to specify the verb it should respond to. Examples:

* CFML:
```cfs
	<cffunction name="getUser" taffy:verb="get">
	    <cfargument name="userId" type="numeric" />
	</cffunction>
```

* CFScript:
```cfs
	function getUser( numeric userId ) taffy:verb="get"
	{
	}
```

<h2 id="representation_metadata">In Representation Classes</h2>

Representation classes are used to take the data provided by a resource and serialize it into a format usable by the web service consumer. A single Representation Class is capable of serializing native data objects (strings, numbers, queries, structures, arrays, etc) into 1-n formats. Typical formats include JSON, XML, or YAML, but are not limited.

<h3 id="taffy_mime">taffy:mime</h3>

By convention, the mime-types supported by your API are determined by the method names in your representation class. (The default representation class supports only JSON.) Each `getAsX` method in your representation class describes a new mime type, defined in two parts: The X portion of the method name determines the extension of the mime type -- which can be appended to the URI as if it were a file, as in: `/artists/42.json` which would use `getAsJson`. Taffy will also return a content-type header with the content type that you supply in the `taffy:mime` metadata property of the `getAsX` method. Typically, this is something like "application/json" or "application/xml". Examples:

* CFML:
```cfs
	<cffunction name="getAsJson" taffy:mime="application/json">
	</cffunction>
```

* CFScript:
```cfs
	function getAsJson() taffy:mime="application/json"
	{
	}
```

<h3 id="taffy_default">taffy:default</h3>

When your API supports more than one data format (i.e. json and xml), you must set one as the default. You do this with the `taffy:default` property, which expects a boolean value of either TRUE or FALSE. The default is FALSE, and you never need to include `taffy:default="false"`. You only need to include `taffy:default="true"` on one method -- the one that should be the default. Examples:

* CFML:
```cfs
	<cffunction name="getAsJson" taffy:mime="application/json" taffy:default="true">
	</cffunction>

	<cffunction name="getAsXml" taffy:mime="application/xml">
	</cffunction>
```

* CFScript:
```cfs
	function getAsJson() taffy:mime="application/json" taffy:default="true"
	{
	}

	function getAsXml() taffy:mime="application/xml"
	{
	}
```
