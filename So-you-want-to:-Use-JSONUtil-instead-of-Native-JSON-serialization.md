_This example is implemented in the folder `examples/api_jsonutil/` (included in the download)._

Using [JSONUtil](http://jsonutil.riaforge.org) instead of the native JSON serialization functionality is as easy as creating a custom representation class that indicates it serializes to the "application/json" mime type and implements the "getAsJson" method:

We've already discussed [the basics of creating a custom representation class](https://github.com/atuttle/Taffy/wiki/So-you-want-to:-Serialize-data-to-a-different-data-type), so here we'll focus on specifically how to use JSONUtil.

**JsonUtilRepresentation.cfc:**
```cfm
<cfcomponent extends="taffy.core.baseRepresentation">

	<cfset variables.jsonUtil = application.jsonUtil />

	<cffunction
		name="getAsJson"
		output="false"
		taffy:mime="application/json"
		taffy:default="true">
			<cfreturn variables.jsonUtil.serialize(variables.data) />
	</cffunction>

</cfcomponent>
```

This example assumes that you've stored an instance of JSONUtil in the Application scope. As discussed in [the xml example](https://github.com/atuttle/Taffy/wiki/So-you-want-to:-Serialize-data-to-a-different-data-type), this is not the best way to write your code, and we have **an example on using dependency injection instead (todo)**, but doing so here keeps it as simple as possible to just illustrate JSONUtil usage and no other "new" concepts.

By setting `taffy:mime="application/json"` and naming the method `getAsJson`, Taffy knows that this class should be used to respond to requests for json data.

Once you've created this custom representation class, which I've named "JsonUtilRepresentation.cfc", you need to tell your application to use it. You do so by adding this code to Application.cfc:

```cfs
function configureTaffy(){
	setDefaultRepresentationClass("JsonUtilRepresentation");
}
```

Notice that I'm not providing some long fully-qualified dot-notation path for the CFC. Instead, I've saved it in my `/resources` folder, and instead we can simply use the file name minus the ".cfc".