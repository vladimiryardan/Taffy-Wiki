>**Warning:** Documentation Currently Outdated<br/><br/>As this product is pre- version 1.0, I am still making breaking changes and not going out of my way to preserve backward compatibility as I come up with better ideas for the ways things can work. The documentation is currently a little bit outdated, but will be updated as part of the 1.0 release.<br/><br/>While the documentation is out of date, the example applications will show working code of varying kinds; as these are my test cases, and I don't develop the core without updating the tests.<br/><br/>_Thanks for your patience as I improve Taffy!_

This page shows how to implement a REST API using Taffy. For an index of documentation, see the [[Home]] page, and for a high level explanation of Taffy, see the [project homepage](http://atuttle.github.com/Taffy/).

By and large, Taffy uses convention over configuration, but there are a few configuration details that can't be pragmatically solved with conventions. In those cases, configuration is needed, but minimal.

# Step 0: Installing Taffy

Simply unzip the `taffy` folder into your webroot. An application-specific mapping to a non-web-accessible folder is not sufficient; however a global mapping in your ColdFusion Administrator will work -- in which case you should map `/taffy` to the location of your unzipped `taffy` folder.

# Step 1: Application.cfc

Much like a [FW/1](http://fw1.riaforge.org/) application, Taffy implements a majority of its logic in Application.cfc, as a base class that the Application.cfc of your application will extend. At a minimum, your APIs Application.cfc needs the following:

```cfs
component extends="taffy.core.api" {

	this.name = hash(getCurrentTemplatePath());

	void function applicationHook(){
		registerMimeType("json", "text/json");
	}

	void function registerURIs(){
		//let taffy know about the cfcs in your api so that it can map URIs to them
		addURI("taffy.example.api.artistCollection");
		addURI("taffy.example.api.artistMember");
		//...
	}

}
```

* Application.cfc extends `taffy.core.api`

* Application name is unique

* DO NOT implement the `onApplicationStart`, `onRequestStart`, or `onRequest` methods; these are used by the framework. `ApplicationHook` is called during `onApplicationStart`, and `requestHook` is called during `onRequestStart`, if you need to add code to these event listeners.

* `applicationHook` method is used to set any application-specific variables, as well as register Mime Types that your API supports (more on this later), and set values for some special url parameters (see: [setDebugKey](http://wiki.github.com/atuttle/Taffy/index-of-api-methods#setDebugKey), [setReloadKey](http://wiki.github.com/atuttle/Taffy/index-of-api-methods#setReloadKey), [setReloadPassword](http://wiki.github.com/atuttle/Taffy/index-of-api-methods#setReloadPassword))

* `registerURIs` is a special method called during framework initialization that inspects the metadata from each of your CFCs and caches some information to enable requests to be properly routed. Inside this function, use the [addURI](http://wiki.github.com/atuttle/Taffy/index-of-api-methods#addURI) method to notify the framework about each CFC in your API. Pass the dot-notation path of the CFC to the `addURI` method.

Of course, there are a few more things you can do. The [[Index of API Methods]] lists all methods, what they do, and where you can use them.

# Step 2: Implement API Resources as CFCs

Each resource in your API (eg. Artists, Art) should be defined as its own CFC. In a Taffy API, **you implement Collections and Members as separate CFCs** (eg. `artistCollection.cfc` and `artistMember.cfc`). They won't be exposed by name via the framework, so you can name them whatever you like. In addition, if you like, your API CFCs may be outside the web-root, as long as they are accessible via CF mappings. Application-specific mappings are acceptable in this case.

Here is an example resource implementation:

**artCollection.cfc:**

```cfs
component extends="taffy.core.restapi" taffy_uri="/artists/{artistId}/art" {

	public function get(string city = ""){
		//query the database for matches, making use of optional parameter "city"
		return representationOf(queryObject).withStatus(200);
	}

}
```

* Each CFC extends `taffy.core.restapi`.

* Each CFC should implement _up to_ 4 methods: **get**, **post**, **put**, and **delete**. As you may have guessed, these map directly to the HTTP verb used by the api consumer.<br/>
  * If the consumer uses the PUT verb, it runs the PUT method in the corresponding CFC. <br/><br/>
  * Since the POST, PUT, and DELETE methods are not implemented in the example above, the POST, PUT, and DELETE verbs will be refused (with HTTP status code 405 Not Allowed) for the corresponding resource.

* Tokens from the component attribute `taffy_uri`, defined as `{token_name}` (including the curly braces, see example above) will be extracted from the URI and passed by name to the requested method. In addition, all query string parameters (except those defined for debugging and reloading) will also be included in the argument collection sent to the method. <br/><br/>For example: `GET /artist/44/art?city=Philadelphia` will result in the GET method being called on the _art collection_, with the arguments: `{ artistId: 44, city: "Philadelphia" }`.<br/><br/>The `artistId` parameter is defined by the `taffy_uri` attribute **of the component (not the function)**, and the `city` parameter is defined as an optional argument to the function, and was provided in the query string.

* The `representationOf` method is a special method provided by the `restapi` base class that creates a new object instance capable of serializing your data into the appropriate format. A generic class capable of serializing as JSON is included with the framework and used by default. To use a custom object, pass the dot-notation CFC path to your custom representation object as the 2nd argument to the `representationOf` method. (It is also planned to implement a global default representation object override, but that is not yet available.) See [[Using a Custom Representation Class]] for more details.<br/><br/>
  ** For the PUT and DELETE verbs, you may choose not to return any data -- in these cases, a status code is usually enough. For those cases, the [noData](http://wiki.github.com/atuttle/Taffy/index-of-api-methods#noData) method is available.<br/><br/>
  ** With either **representationOf** or **noData**, you may optionally use the [withStatus](http://wiki.github.com/atuttle/Taffy/index-of-api-methods#withStatus) method to set the http status code. If you do not include it, a default status of 200 will be returned.

# Step 3: Accessing Your API

Now that you've got a working API, you and your consumers need to know how to access it. This is extremely simple.

The folder containing your API should contain, at a minimum, the **Application.cfc** from Step 1, and an empty **index.cfm**. (index.cfm's contents will be ignored, but it must exist.) To compose a complete URL, append the URI to the location of index.cfm. Assuming your API is located at `http://example.com/api/index.cfm`, and you've implemented the resource with URI `/artists` and the GET method, then you could open up the URL: `http://example.com/api/index.cfm/artists` in your browser and the data would be returned, serialized using the default mime type (JSON unless otherwise defined).

Some people would prefer to remove the `/index.cfm` portion of the URL. To do so, you must use URL-rewriting. **The good news is that you should only need one simple rule.** With Apache you can use [mod_rewrite](http://httpd.apache.org/docs/2.2/mod/mod_rewrite.html), or with IIS 6 you can use [IIRF](http://iirf.codeplex.com/) (free) or [ISAPI Rewrite](http://www.isapirewrite.com/) (paid). IIS 7 has url rewriting built-in. For specific rewriting rule examples for each engine, see [[URL Rewrite Rule Examples]]. There are also Java Servlet Filters to accomplish this, should that be more to your liking.