>*Warning:* Documentation Currently Outdated<br/><br/>As this product is pre- version 1.0, I am still making breaking changes and not going out of my way to preserve backward compatibility as I come up with better ideas for the ways things can work. The documentation is currently a little bit outdated, but will be updated as part of the 1.0 release.<br/><br/>While the documentation is out of date, the example applications will show working code of varying kinds; as these are my test cases, and I don't develop the core without updating the tests.<br/><br/>_Thanks for your patience as I improve Taffy!_

This page is an alphabetical listing of all methods that Taffy exposes for you to use in your APIs. If a method is not included here, you should consider it un-documented, and as such it may change or be renamed at any time without notice. I will make a conscious effort not to break or rename any documented functions, though they may change with a major version if it is an improvement to the framework.

* <a href="#Application_cfc_Methods">Application.cfc Methods</a>
** <a href="#addURI">addURI</a>
** <a href="#applicationStartEvent">applicationStartEvent</a>
** <a href="#configureTaffy">configureTaffy</a>
** <a href="#requestStartEvent">requestStartEvent</a>
** <a href="#registerMimeType">registerMimeType</a>
** <a href="#setDebugKey">setDebugKey</a>
** <a href="#setDefaultMime">setDefaultMime</a>
** <a href="#setReloadKey">setReloadKey</a>
** <a href="#setReloadPassword">setReloadPassword</a>
* <a href="#API_CFC_Methods">API CFC Methods</a>
** <a href="#noData">noData</a>
** <a href="#representationOf">representationOf</a>
** <a href="#withStatus">withStatus</a>

<h2 id="Application_cfc_Methods">Application.cfc Methods</h2>

The following methods are available in your Application.cfc:

<h3 id="addURI"> addURI(string cfcPath)</h3>

*Use it inside:* <a href="#configureTaffy">configureTaffy</a>
*Parameters:*

* cfcPath (string) - The dot-notation path to the cfc that you are notifying the framework of. (e.g. "org.fusiongrokker.api.artGallery.artistCollection")

This method notifies the framework of a CFC that implements part of your API. By using it inside the configureTaffy function, the framework can reload cached objects at the appropriate time. It is not recommended that you use *addURI* outside of *configureTaffy*. If you do not use this method at all, Taffy will not know about any resources your API exposes and thus it can not do anything.

<h3 id="applicationStartEvent">applicationStartEvent()</h3>

*Use it inside:* Application.cfc
*Parameters:* _(none)_

Since the framework takes over the *onApplicationStart* event of Application.cfc, it also exposes this methdod as a way for you to add logic to be executed when the event occurs. *applicationStartEvent* is called by *onApplicationStart* after the framework is initialized. Your Application.cfc *should NOT implement onApplicationStart*, and should instead use *applicationStartEvent*.

<h3 id="configureTaffy">configureTaffy()</h3>

*Use it inside:* Application.cfc
*Parameters:* _(none)_

This is a special method called by the framework during initialization to get your APIs configuration. You must implement this method in your Application.cfc for the framework to work correctly. At a minimum, it should include at least one call to <a href="#addURI">addURI</a>, but all other configuration methods should be used here as well, if you intend to use them.

<h3 id="requestStartEvent">requestStartEvent()</h3>

*Use it inside:* Application.cfc
*Parameters:* _(none)_

Since the framework takes over the *onRequestStart* event of Application.cfc, it also exposes this methdod as a way for you to add logic to be executed when the event occurs. *requestStartEvent* is called by *onRequestStart* after the framework is re-initialized (if requested). Your Application.cfc *should NOT implement onRequestStart*, and should instead use *requestStartEvent*.

<h3 id="registerMimeType">registerMimeType(string extension, string mimeType)</h3>

*Use it inside:* <a href="#configureTaffy">configureTaffy</a>
*Parameters:*

* extension (string) - The url-extension that will invoke this mime type. (e.g. "xml" or "json")
* mimeType (string) - The mime type that should be set into the http headers when the corresponding extension is requested.

Each mime type that your API is capable of returning needs to be registered with the framework, and the *applicationStartEvent* method is the most efficient place to do so.

Because the framework implements JSON by default, the JSON ("application/json") mime type is already registered, and you do not have to re-register it. (You can if you like, though.)

<h3 id="setDebugKey">setDebugKey(string keyName)</h3>

*Use it inside:* <a href="#configureTaffy">configureTaffy</a>
*Parameters:*

* keyName (string) - Name of the url parameter that enables CF Debug Output. (e.g. "debug")

Sometimes it's useful to see ColdFusion's debug output in the results of your API requests during testing. For that reason, you can include this query string parameter to enable debugging on a per-request basis. 

_If you do not change it, the default value is, "debug"._

<h3 id="setDefaultMime">setDefaultMime(string DefaultMimeType)</h3>

*Use it inside:* <a href="#configureTaffy">configureTaffy</a>
*Parameters:*

* DefaultMimeType (string) - The mime type that should be returned when none is specified by the consumer. (e.g. "json")

It is important to note the difference between the mime type and extension. Here, the extension is expected because of its brevity. While the mime type to be returned is "application/json", you should pass the string "json" to the function. If, for example, you registered the mime type of "xml" as "application/xml" (using *registerMimeType*), and you wanted XML to be the default format, you would pass the string "xml" to the *setDefaultMime* function; and it could be specified on the url by appending ".xml" to the URI, before any query string parameters, like so:

@/artists.xml?city=Philadelphia@

_If not implemented, the framework default mime type is JSON ("application/json")._

<h3 id="setReloadKey">setReloadKey(string keyName)</h3>

*Use it inside:* <a href="#configureTaffy">configureTaffy</a>
*Parameters:*

* keyName (string) - Name of the url parameter that requests the framework to be reloaded. (e.g. "reload")

Used in combination with the reload password (see: *setReloadPassword*), the framework will re-initialize itself. During re-initialization, all configuration settings are re-applied and all cached objects are cleared and reloaded. If the value of the key does not match the reload password, a reload will not be performed. This allows you to set a secret password restrict control of reloading your API to trusted parties.

_If you do not change it, the default value is "reload"._

<h3 id="setReloadPassword">setReloadPassword(string Password)</h3>

*Use it inside:* <a href="#configureTaffy">configureTaffy</a>
*Parameters:*

* keyName (string) - Accepted value of the url parameter that requests the framework to be reloaded. (e.g. "true")

Used in combination with the reload key (see: *setReloadKey*), the framework will re-initialize itself. During re-initialization, all configuration settings are re-applied and all cached objects are cleared and reloaded. If the value of the key does not match the reload password, a reload will not be performed. This allows you to set a secret password restrict control of reloading your API to trusted parties.

_If you do not change it, the default value is, "true"._

<h2 id="API_CFC_Methods">API CFC Methods:</h2>

The following methods are available inside each of your API CFCs:

<h3 id="noData">noData()</h3>

*Use it inside:* get, post, put, and delete methods inside your API CFCs.
*Parameters:* _(none)_

This method allows you to specify that there is no data to be returned for the current request. Generally, you would use it in conjunction with the *<a href="#withStatus">withStatus</a>* method to set a specific return status for the request. For example, if the requested resource doesn't exist, you could return a 404 error like so:

@<cfreturn noData().withStatus(404) />@

<h3 id="representationOf">representationOf(any data, [string customRepresentationClass])</h3>

*Use it inside:* get, post, put, and delete methods inside your API CFCs.
*Parameters:*

* data (any) - The data to return to the consumer.
* _optional_ customRepresentationClass (string) - Dot-notation path to a custom CFC that will serialize your results. Defaults to included generic serializer which supports JSON.

Data can be of any type, including complex data types like queries, structures, and arrays, as long as the serializer knows how to serialize them. For more information on using a custom representation class, see [[Using a custom Representation Class]].

<h3 id="withStatus">withStatus(numeric statusCode)</h3>

*Use it inside:* get, post, put, and delete methods inside your API CFCs.
*Parameters:*

* statusCode (numeric) - the "HTTP Status Code":http://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html to return to the consumer.

This special method may _*only*_ be used in conjunction with *<a href="#noData">noData</a>* or *<a href="#representationOf">representationOf</a>*. It sets the HTTP Status Code of the return.

_If you do not specify a return status code, Taffy will always return status code 200 (OK) by default._