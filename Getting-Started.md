>This page gives a brief overview of implementing a REST API using Taffy. For an index of available documentation or for a high level explanation of Taffy, see the [[Home]] page. For in-depth documentation on all available methods, see the [[Index of API Methods]].

**Like videos more than reading?** I presented about REST and Taffy, covering the basics of Taffy and a few more advanced examples at the 2011 [cfObjective](http://www.cfobjective.com) conference. [Watch the recording and get my slides here](???????).

^------ fix link





For the most part, Taffy uses convention over configuration, but there are a few configuration details that can't be pragmatically solved with conventions. In those cases, configuration kept to a minimum [using metadata](/atuttle/Taffy/wiki/Configuration-via-Metadata) where possible.

# Step 0: Installing Taffy

Taffy is tested and supported on Adobe ColdFusion 7, 8, and 9, Railo 3.1, and OpenBD x.x.

Simply unzip the **taffy** folder into your web-root. An _application-specific mapping_ to a non-web-accessible folder is _**not sufficient**_; however a global mapping in your ColdFusion (Railo/OpenBD) Administrator will work -- in which case you should map `/taffy` to the location of your unzipped **taffy** folder.

The code examples below use the ColdFusion 9 script component syntax for its terseness, but this is not a requirement. If you are more comfortable with `<cfml tags>` then you are free to use them.

# Step 1: Application.cfc

Much like a [FW/1](http://github.com/seancorfield/fw1/) application, Taffy implements a majority of its logic in Application.cfc, as a base class that the Application.cfc of your application will extend. At a minimum, your API's Application.cfc needs the following:

```cfs
component 
extends="taffy.core.api"
{

	//the name can be anything you like
	this.name = 'Your_API_App_Name_Here';

	//use this instead of onApplicationStart
	void function applicationStartEvent()
	{
	}
	
	//use this instead of onRequestStart
	void function requestStartEvent()
	{
	}

}
```

* Application.cfc extends `taffy.core.api` -- this is the core functionality of the framework.

* It is _strongly recommended that you **DO NOT**_ implement the `onApplicationStart`, `onRequestStart`, `onRequest`, or `onError` methods; these are used by the framework. If you choose to do so, you should carefully consider the placement of `super.onApplicationStart()`/`super.onRequestStart()`/`super.onRequest()` in your overridden version. Alternatively, Taffy will call `applicationStartEvent` during `onApplicationStart`, and `requestStartEvent` during `onRequestStart`, if you need to add code to these event listeners. The same is true for `onError`, except that there is no pre-determined method to override - you should override `onError` and if necessary, call `super.onError()`.

The [[Index of API Methods]] lists all methods available in Application.cfc, what they do, and where you can use them.

# Step 2: Implement API Resources as CFCs

Each Resource in your API (eg. Person, Product) should be defined as its own CFC. In a Taffy API, **you implement Collections and Members as separate CFCs** (eg. `personCollection.cfc` and `personMember.cfc`, `productCollection.cfc` and `productMember.cfc`). Taffy won't expose them by name, so you can name them whatever you like -- I just happen to like naming them foo**Collection** vs foo**Member** to make it easy to determine which CFC handles individual records (member) and which handles sets of records (collection).

Here is an example resource implementation:

**personCollection.cfc:**

```cfs
component 
extends="taffy.core.resource" 
taffy_uri="/product/{productId}/comments"
{
	public function get(numeric productId, string color = ""){
		//query the database for matches, making use of optional parameter "color"
		//then...
		return representationOf(queryObject).withStatus(200);
	}
}
```

>Note: The namespacing of Taffy's metadata attributes, such as `taffy_uri` is supported using two formats: underscores ("taffy_uri"), and colons ("taffy:uri"). The latter is preferred, but not supported in CF9.01 script component syntax \([ColdFusion Bug #9999]()\), which is why the former was added.

^---- add link to cf bug







* Every Resource CFC -- both member and collection types -- extend `taffy.core.resource`.

* Every Resource CFC should implement _at least 1_ of 4 methods:
  * **get**
  * **post**
  * **put**
  * **delete**
As you may have guessed, these map directly to the HTTP verb used by the API consumer in their request.<br/>
  * If the consumer uses the POST verb, it runs the POST method in the corresponding CFC.<br/><br/>
  * Since the POST, PUT, and DELETE methods are not implemented in the example above, usage of each of the corresponding verbs against the example resource will be refused, with HTTP status code `405 Not Allowed`.

* **Tokens** from the component metadata attribute `taffy:uri` (or `taffy_uri`), defined as `{token_name}` (including the curly braces, see example above) will be extracted from the URI and passed by name to the requested method. In addition, all query string parameters (except those defined for framework specific things like debugging and reloading) will also be included in the argument collection sent to the method. <br/><br/>For example: `GET /product/44?color=Blue` will result in the GET method being called on the _**Product** collection_, with the arguments: `{ productId: 44, color: "Blue" }`.<br/><br/>The `productId` parameter is defined by the `taffy:uri` attribute **(set at the component level, not the function level)**, and the `color` parameter is defined as an optional argument to the function, and was provided in the query string. It is not possible to put optional parameters in the URI -- they need to be in the query string.

* The `representationOf` method -- a special method provided by the `taffy.core.resource` parent class -- creates a new object instance capable of serializing your data into the appropriate format. A class capable of serializing as JSON using ColdFusion's native serialization functionality is included with the framework and used by default. To use a custom representation class on a per-request basis, pass the dot-notation CFC path (or bean id; more on this in [[Bean Factories]]) of your custom representation object as the (optional) 2nd argument to the `representationOf` method. You can also override the global default representation class by using [setDefaultRepresentationClass](/atuttle/Taffy/wiki/Index-of-API-Methods). See [[Using a Custom Representation Class]] for more details on that.<br/><br/>
  * In some cases, you might not want to return any data and a status code is sufficient (for example, indicating a successful delete). For this, the [noData](/atuttle/Taffy/wiki/Index-of-API-Methods) method is available.<br/><br/>
  * With either **representationOf** or **noData**, you may optionally use the [withStatus](/atuttle/Taffy/wiki/Index-of-API-Methods) method to set the HTTP status code, and/or the [withHeaders](/atuttle/Taffy/wiki/Index-of-API-Methods) method to add additional headers to the response. If you do not include `withStatus(...)`, a default status of 200 will be returned. You should familiarize yourself with [[Common API HTTP Status Codes]].

# Step 3: Accessing Your API

Now that you've got a working API, you and your consumers need to know how to access it. This is extremely simple.

The folder containing your API should contain, at a minimum, the **Application.cfc** from Step 1, and an empty **index.cfm**. (index.cfm's contents, if any, will be ignored, but it must exist to allow ColdFusion to handle the request.) To compose a complete URL, append the URI to the location of index.cfm.

Assuming your API is located at `http://example.com/api/index.cfm`, and you've implemented the resource with URI `/products` and the GET method, then you could open up the URL: `http://example.com/api/index.cfm/products` in your browser and the data would be returned, serialized using the default mime type (JSON unless otherwise defined).

Some people would prefer to remove the `/index.cfm` portion of the URL (I am one of them). To do so, you must use URL-rewriting. **The good news is that you should only need one simple rule.** With Apache you can use [mod_rewrite](http://httpd.apache.org/docs/2.2/mod/mod_rewrite.html), or with IIS 6 you can use [IIRF](http://iirf.codeplex.com/) (free) or [ISAPI Rewrite](http://www.isapirewrite.com/) (paid). IIS 7 has url rewriting built-in. For specific rewriting rule examples for each engine, see [[URL Rewrite Rule Examples]]. There are also various Java Servlet Filters to accomplish URL Rewriting, should that be more to your liking.