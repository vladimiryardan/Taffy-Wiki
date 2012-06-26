There's really not much to this, actually. Only a single place where you're _required_ to deviate from "normal" Taffy syntax.

When creating your Resources in CFML (tag) syntax, you use this syntax to define your URI:

```cfm
<cfcomponent taffy:uri="/photos/tagged/portrait">
</cfcomponent>
```

However, there is [a bug in ColdFusion](https://bugbase.adobe.com/index.cfm?event=bug&id=3043394) that prevents you from using a colon in custom metadata attributes **in script components**. So, to work around this, Taffy also allows you to use an underscore in its place:

```cfs
component taffy_uri="/photos/tagged/sunset" {}
```

The exact same issue and resolution applies if any of your Resource Responder methods (for GET, POST, PUT, DELETE, etc) make use of Taffy's custom metadata. For example, if you want to name your method `listPhotos` and have it respond to the GET verb:

```cfm
<cffunction name="listPhotos" taffy:verb="GET"></cffunction>
```
```cfs
function listPhotos() taffy_verb="GET" {}
```