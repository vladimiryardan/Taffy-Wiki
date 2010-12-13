>This page shows how to implement a REST API using Taffy. For an index of available documentation or for a high level explanation of Taffy, see the [[Home]] page.

For the most part, Taffy uses convention over configuration, but there are a few configuration details that can't be pragmatically solved with conventions. In those cases, configuration is needed, but minimal - and usually [accomplished with metadata](/atuttle/Taffy/wiki/Configuration-via-Metadata) to keep things simple and concise.

# Step 0: Installing Taffy

Currently, Taffy is only tested and supported on Adobe ColdFusion 7, 8, and 9. Ultimately, I would like to support Railo and OpenBD. However, for the time being, I've favored getting a release ready over testing against every possible engine.

Simply unzip the **taffy** folder into your web-root. An _application-specific mapping_ to a non-web-accessible folder is not sufficient; however a global mapping in your ColdFusion Administrator will work -- in which case you should map `/taffy` to the location of your unzipped **taffy** folder.

Many of the examples below use the ColdFusion 9 script component syntax for its terseness, but this is not a requirement. If you are more comfortable with `<cf_tags>` then you are free to use them.

# Step 1: Application.cfc

Much like a [FW/1](http://github.com/seancorfield/fw1/) application, Taffy implements a majority of its logic in Application.cfc, as a base class that the Application.cfc of your application will extend. At a minimum, your APIs Application.cfc needs the following:

```cfs
component extends="taffy.core.api" {

	//the name can be anything you like
	this.name = hash(getCurrentTemplatePath());

	//use this instead of onApplicationStart()
	void function applicationStartEvent(){}
	
	//use this instead of onRequestStart()
	void function requestStartEvent(){}

}
```

* Application.cfc extends `taffy.core.api`

* DO NOT implement the `onApplicationStart`, `onRequestStart`, or `onRequest` methods; these are used by the framework. Your implementation of `applicationStartEvent` is called during `onApplicationStart`, and `requestStartEvent` is called during `onRequestStart`, if you need to add code to these event listeners.

* **applicationStartEvent** method is the most appropriate place to set any application-specific variables (e.g. `Application.datasource`).

The [[Index of API Methods]] lists all methods, what they do, and where you can use them.

# Step 2: Implement API Resources as CFCs

Each Resource in your API (eg. Artists, Art) should be defined as its own CFC. In a Taffy API, **you implement Collections and Members as separate CFCs** (eg. `artistCollection.cfc` and `artistMember.cfc`). They won't be exposed by name via the framework, so you can name them whatever you like -- I just happen to like naming them foo**Collection** vs foo**Member** to make it easy to determine which CFC handles individual records (member) and which handles sets of records (collection).

Here is an example resource implementation:

**artCollection.cfc:**

```cfs
component 
	extends="taffy.core.resource" 
	taffy:uri="/artists/{artistId}/art"
{
	public function get(numeric artistId, string city = ""){
		//query the database for matches, making use of optional parameter "city"
		//then...
		return representationOf(queryObject).withStatus(200);
	}
}
```

* Each Resource CFC extends `taffy.core.resource`.

* Each Resource CFC should implement _at least 1_ of 4 methods: **get**, **post**, **put**, and **delete**. As you may have guessed, these map directly to the HTTP verb used by the api consumer to make the request.<br/>
  * If the consumer uses the PUT verb, it runs the PUT method in the corresponding CFC. <br/><br/>
  * Since the POST, PUT, and DELETE methods are not implemented in the example above, the POST, PUT, and DELETE verbs will be refused (with HTTP status code `405 Not Allowed`) for the corresponding resource.

* **Tokens** from the component metadata attribute `taffy:uri`, defined as `{token_name}` (including the curly braces, see example above) will be extracted from the URI and passed by name to the requested method. In addition, all query string parameters (except those defined for framework specific things like debugging and reloading) will also be included in the argument collection sent to the method. <br/><br/>For example: `GET /artist/44/art?city=Philadelphia` will result in the GET method being called on the _ART collection_, with the arguments: `{ artistId: 44, city: "Philadelphia" }`.<br/><br/>The `artistId` parameter is defined by the `taffy:uri` attribute **(set at the component level, not the function level)**, and the `city` parameter is defined as an optional argument to the function, and was provided in the query string.

* The `representationOf` method -- a special method provided by the `taffy.core.resource` parent class -- creates a new object instance capable of serializing your data into the appropriate format. A class capable of serializing as JSON using ColdFusion's native serialization functionality is included with the framework and used by default. To use a custom representation class on a per-request basis, pass the dot-notation CFC path (or bean id; more on this in [[Bean factories]]) of your custom representation object as the (optional) 2nd argument to the `representationOf` method. You can also override the global default representation class by using [setDefaultRepresentationClass](/atuttle/Taffy/wiki/Index-of-API-Methods#setDefaultRepresentationClass). See [[Using a Custom Representation Class]] for more details on that.<br/><br/>
  * In some cases you might not want to return any data, a status code is sufficient. For this, the [noData](/atuttle/Taffy/wiki/index-of-api-methods#noData) method is available.<br/><br/>
  * With either **representationOf** or **noData**, you may optionally use the [withStatus](/atuttle/Taffy/wiki/index-of-api-methods#withStatus) method to set the HTTP status code. If you do not include it, a default status of 200 will be returned. You should familiarize yourself with [[Common API HTTP Status Codes]].

# Step 3: Accessing Your API

Now that you've got a working API, you and your consumers need to know how to access it. This is extremely simple.

The folder containing your API should contain, at a minimum, the **Application.cfc** from Step 1, and an empty **index.cfm**. (index.cfm's contents, if any, will be ignored, but it must exist.) To compose a complete URL, append the URI to the location of index.cfm.

Assuming your API is located at `http://example.com/api/index.cfm`, and you've implemented the resource with URI `/artists` and the GET method, then you could open up the URL: `http://example.com/api/index.cfm/artists` in your browser and the data would be returned, serialized using the default mime type (JSON unless otherwise defined).

Some people would prefer to remove the `/index.cfm` portion of the URL. To do so, you must use URL-rewriting. **The good news is that you should only need one simple rule.** With Apache you can use [mod_rewrite](http://httpd.apache.org/docs/2.2/mod/mod_rewrite.html), or with IIS 6 you can use [IIRF](http://iirf.codeplex.com/) (free) or [ISAPI Rewrite](http://www.isapirewrite.com/) (paid). IIS 7 has url rewriting built-in. For specific rewriting rule examples for each engine, see [[URL Rewrite Rule Examples]]. There are also Java Servlet Filters to accomplish URL Rewriting, should that be more to your liking.