_This example is implemented in the folder `examples/api_twoFormats/` (included in the download)._

In order for your API to be able to respond with multiple formats of the same data, all that you need to have is a single representation class capable of serializing to each of the desired formats. Practically, that means that if your API should be capable of responding with both XML and JSON, you need a representation class ([how do I create a representation class?](https://github.com/atuttle/Taffy/wiki/So-you-want-to:-Serialize-data-to-a-different-data-type)) with `getAsXML` and `getAsJSON` methods:

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

Something a few people have had trouble with was that they thought they had to have different representation classes for each data format. That is not the case. As in the above code, your (single) representation class should be capable of serializing native ColdFusion objects into all of your API's supported data formats.