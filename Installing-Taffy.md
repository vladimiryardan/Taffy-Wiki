# Step 0: Installing Taffy

Taffy is tested and supported on Adobe ColdFusion 7, 8, and 9; as well as Railo 3.2+.

Simply unzip the **taffy** folder into your web-root. An _application-specific mapping_ to a non-web-accessible folder is _**not sufficient**_; however a global mapping in your ColdFusion Administrator will work -- in which case you should map `/taffy` to the location of your unzipped **taffy** folder. If a global mapping is not an option, or you need to support multiple versions of taffy running on the same server, you may either place taffy in the web root, or make it a subfolder of your API.

# Step 1: Write some code

See [Create a dead simple CRUD API](https://github.com/atuttle/Taffy/wiki/So-you-want-to:-Create-a-dead-simple-CRUD-API) for a guide on creating your first API.

# Step 2: Accessing Your API

Now that you've got a working API, you and your consumers need to know how to access it. This is extremely simple.

The folder containing your API should contain, at a minimum, the **Application.cfc** from Step 1, and an empty **index.cfm**. (index.cfm's contents, if any, will be ignored, but it must exist to allow ColdFusion to handle the request.) To compose a complete URL, append the URI to the location of index.cfm.

Assuming your API is located at `http://example.com/api/index.cfm`, and you've implemented the resource with URI `/products` and the GET method, then you could open up the URL: `http://example.com/api/index.cfm/products` in your browser and the data would be returned, serialized using the default mime type (JSON unless otherwise defined).

Some people would prefer to remove the `/index.cfm` portion of the URL (I am one of them). To do so, you must use URL-rewriting. **The good news is that you should only need one simple rule.** With Apache you can use [mod_rewrite](http://httpd.apache.org/docs/2.2/mod/mod_rewrite.html), or with IIS 6 you can use [IIRF](http://iirf.codeplex.com/) (free) or [ISAPI Rewrite](http://www.isapirewrite.com/) (paid). IIS 7 has url rewriting [sort of](http://fusiongrokker.com/post/url-rewriting-for-mango-on-iis7) built-in. For specific rewriting rule examples for each engine, see [[URL Rewrite Rule Examples]]. There are also various Java Servlet Filters to accomplish URL Rewriting, should that be more to your liking.

>## Special requirements for running on Tomcat
>
>If you're using Tomcat, unless it's the version that ships with CF10, you will need to modify Web.xml to add a servlet mapping for every Taffy-powered API that you're running. From what I can tell, most people that use Railo run it on Tomcat. This is not a requirement for Railo compatibility, but it is a requirement for vanilla Tomcat compatibility. The version of Tomcat that ships with ColdFusion 10 has been modified to support what's called "multiple-wildcard filters", but this modification has not yet made its way back to the Tomcat project. [Find more information on this topic, and directions, here](https://groups.google.com/d/msg/taffy-users/Dajw-d30ZQY/6ml-RubGFVAJ).

# Step 3: Tools and Debugging

Not that you'd ever make a typo or have to make a fix to your code, but should that happen you can reinitialize Taffy by simply hitting `http://example.com/api/index.cfm?reload=true`. You can disable this and/or set a stronger password for it in your Application.cfc file. See the [API Docs](https://github.com/atuttle/Taffy/wiki/Index-of-API-Methods) for more details.

Taffy also ships with an awesome(ly helpful) dashboard that contains a mock client and an instant documentation generator. To access it, simply hit `http://example.com/api/index.cfm/?dashboard`. Don't forget to shut this off in a production environment - all of the configuration options are available in the [API Docs](https://github.com/atuttle/Taffy/wiki/Index-of-API-Methods)