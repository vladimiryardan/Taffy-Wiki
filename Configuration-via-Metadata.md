Metadata is used to apply configuration in a concise and elegant manor, where necessity and possibility intersect.

## In Resources

Resources are CFCs that interpret a request and provide or update the requested data.

### taffy:uri

The **taffy:uri** property applies to the `<cfcomponent>` tag or the `component{}` definition. It configures which URIs that the CFC will respond to. There is no limit to the number of tokens that can be used in a single URI, as long as they are not consecutive and are delimited by at least one character (like a slash). Examples:

* CFML:

```xml
	<cfcomponent taffy:uri="/artist/{artistId}">
	</cfcomponent>
```

* CFScript:

```javascript
	component 
	taffy_uri="/artist/{artistId}"
	{
	}
```


The following example is _invalid_, because tokens are not delimited by a slash:<br/>

```javascript
	taffy:uri="/artist/{artistId}{artistName}"
```


#### Tokens

Tokens in URIs define the parts of the URI that are dynamic. They are identified by curly-brackets: `{tokenName}` and are passed **by name** to all functions that handle the request. The `{foo}` token's value will be passed to the `foo` argument of the associated method.

#### URI Matching Order

As of Taffy 1.3, URIs are processed into a sorted array on API startup, and searched in order for every request, such that `/artists/list` will match before `/artists/{id}`. You should design your API URI's accordingly. Pay special attention to placement and possible values for URI tokens. If a possible value is the same as a static URI, consider changing URI formatting. For example, if we had an artist with id "list", then having the static URI `/artists/list` would prevent the `/artists/{id}` URI from ever matching when the intention is to get the individual artist record for artist with id "list."

### taffy:verb

By convention, resources will automatically map the 4 primary HTTP REST verbs -- GET, PUT, POST, DELETE -- to CFC methods with the same name. For extended verbs -- OPTIONS, HEAD -- or if you want to map one of the primary verbs to a method with a different name, you can use the `taffy:verb` metadata property on the method to specify the verb it should respond to. Examples:

* CFML:

```xml
	<cffunction name="getUser" taffy:verb="get">
	    <cfargument name="userId" type="numeric" />
	</cffunction>
```

* CFScript:

```javascript
	function getUser( numeric userId ) taffy_verb="get"
	{
	}
```

## In Representation Classes

Representation classes are used to take the data provided by a resource and serialize it into a format usable by the web service consumer. A single Representation Class is capable of serializing native data objects (strings, numbers, queries, structures, arrays, etc) into 1 or more formats. Typical formats include JSON, XML, or YAML, but are not limited.

### taffy:mime

By convention, the mime-types supported by your API are determined by the method names in your default representation class. (The included default representation class supports only JSON.) Each `getAsX` method in your representation class describes a new mime type, defined in two parts: The X portion of the method name determines the extension of the mime type -- which can be appended to the URI as if it were a file, as in: `/artists/42.json` which would use `getAsJson`. Taffy will also return a content-type header with the content type that you supply in the `taffy:mime` metadata property of the `getAsX` method. Typically, this is something like "application/json" or "application/xml". Examples:

* CFML:

```xml
	<cffunction name="getAsJson" taffy:mime="application/json">
	</cffunction>
```

* CFScript:

```javascript
	function getAsJson() taffy_mime="application/json"
	{
	}
```

### taffy:default

When your API supports more than one data format (i.e. json and xml), you must set one as the default. You do this with the `taffy:default` property, which expects a boolean value of either TRUE or FALSE. The default is FALSE, and you never need to include `taffy:default="false"`. You only need to include `taffy:default="true"` on one method -- the one that should be the default. Examples:

* CFML:

```xml
	<cffunction name="getAsJson" taffy:mime="application/json" taffy:default="true">
	</cffunction>

	<cffunction name="getAsXml" taffy:mime="application/xml">
	</cffunction>`
```

* CFScript:

```javascript
	function getAsJson() taffy_mime="application/json" taffy_default="true"
	{
	}

	function getAsXml() taffy_mime="application/xml"
	{
	}
```