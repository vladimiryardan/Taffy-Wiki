> This page gives a brief overview of implementing a REST API using Taffy. For an index of available documentation or for a high level explanation of Taffy, see the [[Home]] page. For in-depth documentation on all available methods, see the [[Index of API Methods]].

**Like videos more than reading?** I presented about REST and Taffy, covering the basics of Taffy and a few more advanced examples at the 2011 [cfObjective](http://www.cfobjective.com) conference. [Watch the recording and get my slides here](http://fusiongrokker.com/post/my-cfobjective-2011-slides-notes).

For the most part, Taffy uses convention over configuration, but there are a few configuration details that can't be pragmatically solved with conventions. In those cases, configuration is kept to a minimum [using metadata](/atuttle/Taffy/wiki/Configuration-via-Metadata) wherever possible.

# Step 0: Installing Taffy

Taffy is tested and supported on Adobe ColdFusion 7, 8, and 9; as well as Railo 3.2+.

Simply unzip the **taffy** folder into your web-root. An _application-specific mapping_ to a non-web-accessible folder is _**not sufficient**_; however a global mapping in your ColdFusion Administrator will work -- in which case you should map `/taffy` to the location of your unzipped **taffy** folder. If a global mapping is not an option, or you need to support multiple versions of taffy running on the same server, instead of placing taffy in the web root, you may also make it a subfolder of your API.

The code examples below use the ColdFusion 9 script component syntax for its terseness, but this is not a requirement. If you are more comfortable with `<cfml tags>` then you are free to use them instead.

# Step 1: Application.cfc

Much like a [FW/1](http://github.com/seancorfield/fw1/) application, Taffy implements a majority of its logic in Application.cfc, as a base class that the Application.cfc of your API will extend. At a minimum, your API's Application.cfc needs the following:

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

**What's going on here?**

* Application.cfc extends `taffy.core.api` -- this is most of what makes Taffy work.

* It is _strongly recommended that you **DO NOT**_ implement the `onApplicationStart`, `onRequestStart`, `onRequest`, or `onError` methods; these are used by the framework. If you choose to do so, you should carefully consider the placement of `super.onApplicationStart()`/`super.onRequestStart()`/`super.onRequest()` in your overridden version. Alternatively, Taffy will call `applicationStartEvent` during `onApplicationStart`, and `requestStartEvent` during `onRequestStart`, if you need to add code to these event listeners. The same is true for `onError`, except that there is no pre-determined method to override - you should override `onError` and if necessary, call `super.onError()`.

The [[Index of API Methods]] lists all methods available in Application.cfc, what they do, and where you can use them.

# Step 2: Implement API Resources as CFCs

Each Resource in your API (eg. Person, Product) should be defined as its own CFC. In a Taffy API, **you implement Collections and Members as separate CFCs** (eg. `personCollection.cfc` and `personMember.cfc`, `productCollection.cfc` and `productMember.cfc`). Taffy won't expose them by name, so you can name them whatever you like -- I just happen to like naming them foo**Collection** vs foo**Member** to make it easy to determine which CFC handles individual records (member) and which handles sets of records (collection).

When using the default bean factory (the default configuration), your resource CFCs are required to be in the `/resources` folder. You can use an application-specific mapping, or a server-wide mapping to point `/resources` to the folder containing your resource CFCs, or simply store them in the physical `/resources` path. It can be in the web root, or even the same folder as your Application.cfc (if in a sub-folder of the web root), but it must be named `resources`.

Here is an example resource implementation:

**personCollection.cfc:**

```cfs
component
extends="taffy.core.resource"
taffy_uri="/people"
{
	public function get(string eyeColor = ""){
		//query the database for matches, making use of optional parameter "eyeColor" if provided
		//then...
		return representationOf(someCollectionObject).withStatus(200); //collection might be query, array, etc
	}
}
```

This resource will respond for `http://example.com/api/people`, as well as `http://example.com/api/people?eyeColor=green`. When the query string parameters are provided, they will be passed to the function by name.

And here's a similar implementation of a member for the same data type (person):

**personMember.cfc:**

```cfs
component
extends="taffy.core.resource"
taffy_uri="/people/{personName}"
{
	public function get(string personName){
		//find the requested person, by name
		//then...
		return representationOf(someMemberObject).withStatus(200); //member might be a structure, ORM entity, etc
	}
}
```

This resource will respond for `http://example.com/api/people/john-smith`. For a member GET, the identifying information --name, in this case-- is usually unique (enforced by your database indexes and app logic), so there is rarely a reason to include query string parameters as optional arguments. That said, it is supported.

 * Every Resource CFC -- both member and collection types -- extend `taffy.core.resource`.

 * Every Resource CFC should implement _at least 1_ of 4 methods. As you may have guessed, these map directly to the HTTP verb used by the API consumer in their request.
  * **get**
  * **post**
  * **put**
  * **delete**<br/><br/>

 * If the consumer uses the POST verb, it runs the POST method in the corresponding CFC.
   * Since the POST, PUT, and DELETE methods are not implemented in the example above, usage of each of the corresponding verbs against the example resource will be refused, with HTTP status code `405 Not Allowed`.<br/><br/>

 * **Tokens** from the component metadata attribute `taffy:uri` (or `taffy_uri`), defined as `{token_name}` (including the curly braces, see example above) will be extracted from the URI and passed by name to the requested method. In addition, all query string parameters (except those defined for framework specific things like debugging and reloading) will also be included in the argument collection sent to the method. <br/><br/>For example: `GET /product/44?color=Blue` will result in the GET method being called on the _**Product** collection_, with the arguments: `{ productId: 44, color: "Blue" }`.<br/><br/>The `productId` parameter is defined by the `taffy:uri` attribute **(set at the component level, not the function level)**, and the `color` parameter is defined as an optional argument to the function, and was provided in the query string. It is not possible to put optional parameters in the URI -- they need to be in the query string.

 * The `representationOf` method -- a special method provided by the `taffy.core.resource` parent class -- creates a new object instance capable of serializing your data into the appropriate format. A class capable of serializing as JSON using ColdFusion's native serialization functionality is included with the framework and used by default. You can override the global default representation class by using `variables.framework.representationClass`. See [[Using a Custom Representation Class]] for more details on that.
   * In some cases, you might not want to return any data and a status code is sufficient (for example, indicating a successful delete). In this case, use [noData()](/atuttle/Taffy/wiki/Index-of-API-Methods) instead of `representationOf()`.<br/><br/>
   * With either **representationOf** or **noData**, you may optionally chain the [withStatus](/atuttle/Taffy/wiki/Index-of-API-Methods) method to set the HTTP status code, and/or the [withHeaders](/atuttle/Taffy/wiki/Index-of-API-Methods) method to add additional headers to the response. If you do not include `withStatus(...)`, a default status of 200 will be returned. Chaining looks like this: `return noData().withStatus(...).withHeaders(...);` You should familiarize yourself with [[Common API HTTP Status Codes]].

## A quick Aside

**Note:** The namespacing of Taffy's metadata attributes, such as `taffy_uri` is supported using two formats: underscores ("taffy_uri"), and colons ("taffy:uri"). The latter is my preferred style, but not supported in CF9.01 script component syntax ([ColdFusion Bug #3043656](https://bugbase.adobe.com/index.cfm?event=bug&id=3043656)), which is why the former was added. However, if you're writing your components with tags, the colon-syntax is supported.

# Step 3: Accessing Your API

Now that you've got a working API, you and your consumers need to know how to access it. This is extremely simple.

The folder containing your API should contain, at a minimum, the **Application.cfc** from Step 1, and an empty **index.cfm**. (index.cfm's contents, if any, will be ignored, but it must exist to allow ColdFusion to handle the request.) To compose a complete URL, append the URI to the location of index.cfm.

Assuming your API is located at `http://example.com/api/index.cfm`, and you've implemented the resource with URI `/products` and the GET method, then you could open up the URL: `http://example.com/api/index.cfm/products` in your browser and the data would be returned, serialized using the default mime type (JSON unless otherwise defined).

Some people would prefer to remove the `/index.cfm` portion of the URL (I am one of them). To do so, you must use URL-rewriting. **The good news is that you should only need one simple rule.** With Apache you can use [mod_rewrite](http://httpd.apache.org/docs/2.2/mod/mod_rewrite.html), or with IIS 6 you can use [IIRF](http://iirf.codeplex.com/) (free) or [ISAPI Rewrite](http://www.isapirewrite.com/) (paid). IIS 7 has url rewriting [sort of](http://fusiongrokker.com/post/url-rewriting-for-mango-on-iis7) built-in. For specific rewriting rule examples for each engine, see [[URL Rewrite Rule Examples]]. There are also various Java Servlet Filters to accomplish URL Rewriting, should that be more to your liking.

## Special requirements for running on Tomcat

From what I can tell, most people that use Railo run it on Tomcat. This is not a requirement for Railo compatibility, but it is a requirement for vanilla Tomcat compatibility. The version of Tomcat that ships with ColdFusion 10 has been modified to support what's called "multiple-wildcard filters", but this modification has not yet made its way back to the Tomcat project. This means that if you're using Tomcat, unless it's the version that ships with CF10, you will need to modify Web.xml to add a servlet mapping for every Taffy-powered API that you're running. (Not every resource, just one per API.) [Find more information on this topic, and directions, here](https://groups.google.com/d/msg/taffy-users/Dajw-d30ZQY/6ml-RubGFVAJ).

# Step 4: Tools and Debugging

Not that you'd ever make a typo or have to make a fix to your code, but should that happen you can reinitialize Taffy by simply hitting `http://example.com/api/index.cfm?reload=true`. You can disable this and/or set a stronger password for it in your Application.cfc file. See the [API Docs](https://github.com/atuttle/Taffy/wiki/Index-of-API-Methods) for more details.

Taffy also ships with an awesome(ly helpful) dashboard that contains a mock client and an instant documentation generator. To access it, simply hit `http://example.com/api/index.cfm/?dashboard`. As of Taffy 1.3, `?dashboard` is deprecated and you should just hit the root of your API: `http://example.com/api/`. Don't forget to shut this off in a production environment - all of the configuration options are available in the [API Docs](https://github.com/atuttle/Taffy/wiki/Index-of-API-Methods)