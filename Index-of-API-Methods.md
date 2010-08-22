This page is an alphabetical listing of all methods that Taffy exposes for you to use in your APIs. If a method is not included here, you should consider it un-documented, and as such it may change or be renamed at any time without notice. I will make a conscious effort not to break or rename any documented functions, though they may change with a major version if it is an improvement to the framework.

* [Application.cfc Methods](#Application_cfc_Methods)
  * [addURI](#addURI)
  * [applicationStartEvent](#applicationStartEvent)
  * [configureTaffy](#configureTaffy)
  * [requestStartEvent](#requestStartEvent)
  * [registerMimeType](#registerMimeType)
  * [setDashboardKey](#setDashboardKey)
  * [setDebugKey](#setDebugKey)
  * [setDefaultMime](#setDefaultMime)
  * [setReloadKey](#setReloadKey)
  * [setReloadPassword](#setReloadPassword)
* [Resource CFC Methods](#Resource_CFC_Methods)
  * [noData](#noData)
  * [representationOf](#representationOf)
  * [withStatus](#withStatus)

<h2 id="Application_cfc_Methods">Application.cfc Methods</h2>

The following methods are available in your Application.cfc:

<h3 id="applicationStartEvent">applicationStartEvent()</h3>

**Use it inside:** Application.cfc
**Parameters:** _(none)_

Since the framework takes over the **onApplicationStart** event of Application.cfc, it also exposes this methdod as a way for you to add logic to be executed when the event occurs. **applicationStartEvent** is called by **onApplicationStart** before the framework is initialized. Your Application.cfc **should NOT implement onApplicationStart**, and should instead use **applicationStartEvent**.

<h3 id="configureTaffy">configureTaffy()</h3>

**Use it inside:** Application.cfc
**Parameters:** _(none)_

This is a special method called by the framework during initialization to get your APIs configuration. You must implement this method in your Application.cfc for the framework to work correctly. All configuration methods should be used inside your implementation of **configureTaffy**, if you intend to use them.

<h3 id="requestStartEvent">requestStartEvent()</h3>

**Use it inside:** Application.cfc
**Parameters:** _(none)_

Since the framework takes over the **onRequestStart** event of Application.cfc, it also exposes this methdod as a way for you to add logic to be executed when the event occurs. **requestStartEvent** is called by **onRequestStart** after the framework is re-initialized (if requested). Your Application.cfc **should NOT implement onRequestStart**, and should instead use **requestStartEvent**.

<h3 id="registerMimeType">registerMimeType(string extension, string mimeType)</h3>

**Use it inside:** [configureTaffy](#configureTaffy)
**Parameters:**

* extension (string) - The url-extension that will invoke this mime type. (e.g. "xml" or "json")
* mimeType (string) - The mime type that should be set into the http headers when the corresponding extension is requested. (e.g. "application/xml" or "application/json")

Each mime type that your API is capable of returning needs to be registered with the framework, and the **configureTaffy** method is the most efficient place to do so.

Because the framework implements JSON by default, the JSON ("application/json") mime type is already registered, and you do not have to re-register it. (You can if you like, though. It won't hurt anything.)

<h3 id="setDebugKey">setDebugKey(string keyName)</h3>

**Use it inside:** [configureTaffy](#configureTaffy)
**Parameters:**

* keyName (string) - Name of the url parameter that enables CF Debug Output. (e.g. "debug")

Sometimes it's useful to see ColdFusion's debug output in the results of your API requests during testing. For that reason, you can include this query string parameter to enable debugging on a per-request basis. 

_If you do not change it, the default value is, "debug"._

<h3 id="setDefaultMime">setDefaultMime(string DefaultMimeType)</h3>

**Use it inside:** [configureTaffy](#configureTaffy)
**Parameters:**

* DefaultMimeType (string) - The mime type that should be returned when none is specified by the consumer. (e.g. "json")

It is important to note the difference between the mime type and extension. Here, the extension is expected because of its brevity. While the mime type to be returned is "application/json", you should pass the string "json" to the function. If, for example, you registered the mime type of "xml" as "application/xml" (using **registerMimeType**), and you wanted XML to be the default format, you would pass the string "xml" to the **setDefaultMime** function; and it could be specified on the url by appending ".xml" to the URI, before any query string parameters, like so:

`/artists.xml?city=Philadelphia`

_If not implemented, the framework default mime type is JSON ("application/json")._

<h3 id="setReloadKey">setReloadKey(string keyName)</h3>

**Use it inside:** [configureTaffy](#configureTaffy)
**Parameters:**

* keyName (string) - Name of the url parameter that requests the framework to be reloaded. (e.g. "reload")

Used in combination with the reload password (see: **setReloadPassword**), the framework will re-initialize itself. During re-initialization, all configuration settings are re-applied and all cached objects are cleared and reloaded. If the value of the key does not match the reload password, a reload will not be performed. This allows you to set a secret password to restrict control of reloading your API to trusted parties.

_If you do not change it, the default value is "reload"._

<h3 id="setReloadPassword">setReloadPassword(string Password)</h3>

**Use it inside:** [configureTaffy](#configureTaffy)
**Parameters:**

* keyName (string) - Accepted value of the url parameter that requests the framework to be reloaded. (e.g. "true")

Used in combination with the reload key (see: **setReloadKey**), the framework will re-initialize itself. During re-initialization, all configuration settings are re-applied and all cached objects are cleared and reloaded. If the value of the key does not match the reload password, a reload will not be performed. This allows you to set a secret password to restrict control of reloading your API to trusted parties.

_If you do not change it, the default value is, "true"._

<h2 id="Resource_CFC_Methods">Resource CFC Methods:</h2>

The following methods are available inside each of your Resource CFCs:

<h3 id="noData">noData()</h3>

**Use it inside:** get, post, put, and delete methods inside your Resource CFCs.
**Parameters:** _(none)_

This method allows you to specify that there is no data to be returned for the current request. Generally, you would use it in conjunction with the **[withStatus](#withStatus)** method to set a specific return status for the request. For example, if the requested resource doesn't exist, you could return a 404 error like so:

```cfs
return noData().withStatus(404);
```

<h3 id="representationOf">representationOf(any data, [string customRepresentationClass])</h3>

**Use it inside:** get, post, put, and delete methods inside your Resource CFCs.
**Parameters:**

* data (any) - The data to return to the consumer.
* _optional_ customRepresentationClass (string) - Dot-notation path to a custom CFC that will serialize your results. Defaults to included generic serializer which supports JSON.

Data can be of any type, including complex data types like queries, structures, and arrays, as long as the serializer knows how to serialize them. For more information on using a custom representation class, **see [[Using a custom Representation Class]]**.

<h3 id="withStatus">withStatus(numeric statusCode)</h3>

**Use it inside:** get, post, put, and delete methods inside your Resource CFCs.
**Parameters:**

* statusCode (numeric) - the [HTTP Status Code](http://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html) to return to the consumer.

This special method may _**only**_ be used in conjunction with **[noData](#noData)** or **[representationOf](#representationOf)**. It sets the HTTP Status Code of the return.

_If you do not specify a return status code, Taffy will always return status code 200 (OK) by default._