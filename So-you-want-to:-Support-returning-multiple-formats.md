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

## A Note on Supporting Multiple Formats

When your API supports multiple return formats, you should be aware of what your default format is, and the precedence if there is a conflict between two different requested formats for the same request. It is possible to request the format via the URI ("foo.json"), as well as via a request header ("ACCEPT: application/xml"). When both methods are used in the same request, which takes precedence?

In Taffy, the value specified in the URI takes precedence. While this may seem unintuitive because the header would take more effort, and thus explicitness and intent to send, the URI method provides more utility because it allows clients with less capability to accomplish the same goal. For example, what if your API supports image return types? You can't specify request-headers for an `<img/>` tag.

Both are supported out of the box, but you should be aware that URL formats take precedence over headers.