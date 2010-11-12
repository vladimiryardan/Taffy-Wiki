This page is an alphabetical listing of all methods that Taffy exposes for you to use in your APIs. If a method is not included here, you should consider it un-documented, and as such it may change or be renamed at any time without notice. I will make a conscious effort not to break or rename any documented functions, though they may change with a major version if it is an improvement to the framework.

* [Application.cfc Methods](#Application_cfc_Methods)
  * [applicationStartEvent](#applicationStartEvent)
  * [configureTaffy](#configureTaffy)
  * [enableDashboard](#enableDashboard)
  * [getPath](#getPath)
  * [newRepresentation](#newRepresentation)
  * [onTaffyRequest](#onTaffyRequest)
  * [requestStartEvent](#requestStartEvent)
  * [registerMimeType](#registerMimeType)
  * [setBeanFactory](#setBeanFactory)
  * [setDashboardKey](#setDashboardKey)
  * [setDebugKey](#setDebugKey)
  * [setDefaultMime](#setDefaultMime)
  * [setDefaultRepresentationClass](#setDefaultRepresentationClass)
  * [setReloadKey](#setReloadKey)
  * [setReloadPassword](#setReloadPassword)
* [Resource CFC Methods](#Resource_CFC_Methods)
  * [noData](#noData)
  * [representationOf](#representationOf)
  * [withHeaders](#withHeaders)
  * [withStatus](#withStatus)

<h2 id="Application_cfc_Methods">Application.cfc Methods</h2>

The following methods are available in your Application.cfc:

<h3 id="applicationStartEvent">applicationStartEvent()</h3>

**Use it inside:** Application.cfc<br/>
**Parameters:** _(none)_

Since the framework takes over the **onApplicationStart** event of Application.cfc, it also exposes this methdod as a way for you to add logic to be executed when the event occurs. **applicationStartEvent** is called by **onApplicationStart** before the framework is initialized. Your Application.cfc **should NOT implement onApplicationStart**, and should instead use **applicationStartEvent**.

<h3 id="configureTaffy">configureTaffy()</h3>

**Use it inside:** Application.cfc<br/>
**Parameters:** _(none)_

This is a special method called by the framework during initialization to get your APIs configuration. You must implement this method in your Application.cfc to override any framework setting defaults. All configuration methods should be used inside your implementation of **configureTaffy**, if you intend to use them.

<h3 id="enableDashboard">enableDashboard(boolean enabled)</h3>

**Use it inside:** [configureTaffy](#configureTaffy)<br/>
**Parameters:**

* enabled (boolean) - Whether or not Taffy will allow the dashboard to be displayed. If set to false, the [dashboard key](#setDashboardKey) is simply ignored.

_Default value is TRUE._

<h3 id="getPath">getPath()</h3>

**Use it inside:** Application.cfc<br/>
**Parameters:** _(none)_

This method is provided as an extension point. On Adobe ColdFusion ("ACF") 9, installed in standard entire-server mode, no change should be necessary. However, if ACF is installed on another JEE app server (i.e. Tomcat, Glassfish, etc), or on JRun but using an EAR/WAR setup, then you may need to override this method to make Taffy work on your server. {link to additional wiki page to document each configuration here}.

<h3 id="newRepresentation">newRepresentation(string class)</h3>

**Use it inside:** [onTaffyRequest](#onTaffyRequest)<br/>
**Parameters:**

* class (string) - the dot-notation cfc path (or bean id if managing the class with a bean factory, including the built-in factory) for the representation class. Optional. Default value is the default representation class.

Use this method inside onTaffyRequest when you want to abort the request with some specific message or data. Use the `setData` method on the returned representation class to put your data/message into it before returning it.

<h3 id="onTaffyRequest">onTaffyRequest(string verb, string cfc, struct requestArguments, string mimeExt, struct headers)</h3>

**Use it inside:** Application.cfc<br/>
**Parameters:**

* verb (string) - The HTTP request verb provided by the consumer
* cfc (string) - The CFC name (minus ".cfc") that would handle the request. (Bean Name, if using an external bean factory.)
* requestArguments (struct) - A structure containing all of the arguments of the request, including tokens from the URI as well as any query string parameters (defined after the ?, eg ?city=).
* mimeExt (string) - The mime extension (e.g. "json" - NOT the full mime type, e.g. "application/json")
* headers (struct) - A structure containing each header from the request, as sent by the consumer.

This method is optional, and allows you to inspect and potentially abort an API request in a way that adheres to the HTTP specification. If you choose not to override it (by implementing it in your Application.cfc), it will always return true, allowing the request to continue. If you implement it, you can check for things like an API key, or whether or not the customer has paid for your service, and return something other than the data that they are requesting.

If you do not return TRUE, allowing the request to continue as normal, then Taffy expects you to return a **[representation](http://github.com/atuttle/Taffy/wiki/Using-a-Custom-Representation-Class)** (either the generic class, included, or a custom one) that it should immediately return to the consumer, serialized to the appropriate format. If you simply want to return with a status code of 403 (which indicates "Not Allowed"), you could do this:

```cfs
return createObject("component", "taffy.core.nativeJsonRepresentation").noData().withStatus(403);
```

Alternately, you could indicate that they owe you money:

```cfs
return createObject("component", "taffy.core.nativeJsonRepresentation").setData({error="Your account is past due. Please email accounts payable."}).withStatus(403);
```

The options here are basically unlimited.

<h3 id="requestStartEvent">requestStartEvent()</h3>

**Use it inside:** Application.cfc<br/>
**Parameters:** _(none)_

Since the framework takes over the **onRequestStart** event of Application.cfc, it also exposes this methdod as a way for you to add logic to be executed when the event occurs. **requestStartEvent** is called by **onRequestStart** after the framework is re-initialized (if requested). Your Application.cfc **should NOT implement onRequestStart**, and should instead use **requestStartEvent**.

<h3 id="registerMimeType"><em>registerMimeType(string extension, string mimeType)</em> (deprecated)</h3>

**This method is deprecated. It will likely be removed in a future update. [Use metadata instead](/atuttle/Taffy/wiki/Configuration-via-Metadata).**

**Use it inside:** [configureTaffy](#configureTaffy)<br/>
**Parameters:**

* extension (string) - The url-extension that will invoke this mime type. (e.g. "xml" or "json")
* mimeType (string) - The mime type that should be set into the http headers when the corresponding extension is requested. (e.g. "application/xml" or "application/json")

Each mime type that your API is capable of returning needs to be registered with the framework, and the **[configureTaffy](#configureTaffy)** method is the most efficient place to do so.

Because the framework implements JSON by default, the JSON ("application/json") mime type is already registered, and you do not have to re-register it. (You can if you like, though. It won't hurt anything.)

<h3 id="setBeanFactory">setBeanFactory(object beanFactory, [string beanList])</h3>
**Use it inside:** [configureTaffy](#configureTaffy)<br/>
**Parameters:**

* beanFactory (object) - already instantiated and cached (e.g. in Application scope) object instance of your external bean factory.
* beanList (string) - a comma-delimited list of bean Id's that the bean factory is aware of. Optional, and only necessary if the beanFactory does not implement `getBeanList()` _and is not ColdSpring_. Tight ColdSpring integration is baked in.

While Taffy includes a very basic Bean Factory (used by default) and also has fairly tight integration with ColdSpring, other bean factories could be added. If your bean factory implements `getBeanList()`, then Taffy will use that to get a list of beans that the factory knows about. However, if it does not, then Taffy will allow you to provide the bean list as a string in the optional second argument.

<h3 id="setDashboardKey">setDashboardKey(string keyName)</h3>

**Use it inside:** [configureTaffy](#configureTaffy)<br/>
**Parameters:**

* keyName (string) - Name of the url parameter that displays the dashboard. Default value is "dashboard".

The dashboard displays resources that your API is aware of, generates documentation about your API based on **hint** attributes, and contains a mock client to make testing your API easy.

<h3 id="setDebugKey">setDebugKey(string keyName)</h3>

**Use it inside:** [configureTaffy](#configureTaffy)<br/>
**Parameters:**

* keyName (string) - Name of the url parameter that enables CF Debug Output. (e.g. "debug")

Sometimes it's useful to see ColdFusion's debug output in the results of your API requests during testing. For that reason, you can include this query string parameter to enable debugging on a per-request basis. 

_If you do not change it, the default value is, "debug"._

<h3 id="setDefaultMime"><em>setDefaultMime(string DefaultMimeType)</em> (deprecated)</h3>

**This method is deprecated. It will likely be removed in a future update. [Use metadata instead](/atuttle/Taffy/wiki/Configuration-via-Metadata).**

**Use it inside:** [configureTaffy](#configureTaffy)<br/>
**Parameters:**

* DefaultMimeType (string) - The mime type that should be returned when none is specified by the consumer. (e.g. "json")

It is important to note the difference between the mime type and extension. Here, the extension is expected because of its brevity. While the mime type to be returned is "application/json", you should pass the string "json" to the function. If, for example, you registered the mime type of "xml" as "application/xml" (using **[registerMimeType](#registerMimeType)**), and you wanted XML to be the default format, you would pass the string "xml" to the **[setDefaultMime](#setDefaultMime)** function; and it could be specified on the url by appending ".xml" to the URI, before any query string parameters, like so:

`/artists.xml?city=Philadelphia`

_If not implemented, the framework default mime type is JSON ("application/json")._

<h3 id="setDefaultRepresentationClass">setDefaultRepresentationClass(string customClassDotPath)</h3>

**Use it inside:** [configureTaffy](#configureTaffy)<br/>
**Parameters:**

* customClassDotPath (string) - Dot-notation path to the CFC to be used by default. Alternately, you can pass in a beanId and the class will be requested from the bean factory. This works with both the internal bean factory (your `/resources` folder) and if you are using an external bean factory.

When you change the default Representation Class, all responses will use your custom default class, unless you specifically override that request, using the (optional) second parameter to the **[representationOf](#representationOf)** function.

<h3 id="setReloadKey">setReloadKey(string keyName)</h3>

**Use it inside:** [configureTaffy](#configureTaffy)<br/>
**Parameters:**

* keyName (string) - Name of the url parameter that requests the framework to be reloaded. (e.g. "reload")

Used in combination with the reload password (see: **[setReloadPassword](#setReloadPassword)**), the framework will re-initialize itself. During re-initialization, all configuration settings are re-applied and all cached objects are cleared and reloaded. If the value of the key does not match the reload password, a reload will not be performed. This allows you to set a secret password to restrict control of reloading your API to trusted parties.

_If you do not change it, the default value is "reload"._

<h3 id="setReloadPassword">setReloadPassword(string Password)</h3>

**Use it inside:** [configureTaffy](#configureTaffy)<br/>
**Parameters:**

* keyName (string) - Accepted value of the url parameter that requests the framework to be reloaded. (e.g. "true")

Used in combination with the reload key (see: **[setReloadKey](#setReloadKey)**), the framework will re-initialize itself. During re-initialization, all configuration settings are re-applied and all cached objects are cleared and reloaded. If the value of the key does not match the reload password, a reload will not be performed. This allows you to set a secret password to restrict control of reloading your API to trusted parties.

_If you do not change it, the default value is, "true"._

<h2 id="Resource_CFC_Methods">Resource CFC Methods:</h2>

Resource CFCs extend `taffy.core.resource`.  The following methods are available inside each of your Resource CFCs:

<h3 id="noData">noData()</h3>

**Use it inside:** responder methods inside your Resource CFCs (e.g. get, put, post, delete - as well as head, options, etc).<br/>
**Parameters:** _(none)_

This method allows you to specify that there is no data to be returned for the current request. Generally, you would use it in conjunction with the **[withStatus](#withStatus)** method to set a specific return status for the request. For example, if the requested resource doesn't exist, you could return a 404 error like so:

```cfs
return noData().withStatus(404);
```

<h3 id="representationOf">representationOf(any data, [string customRepresentationClass])</h3>

**Use it inside:** responder methods inside your Resource CFCs (e.g. get, put, post, delete - as well as head, options, etc).<br/>
**Parameters:**

* data (any) - The data to return to the consumer.
* _optional_ customRepresentationClass (string) - Dot-notation path to a custom CFC that will serialize your results. Defaults to included generic serializer which supports JSON.

Data can be of any type, including complex data types like queries, structures, and arrays, as long as the serializer knows how to serialize them. For more information on using a custom representation class, **see [[Using a custom Representation Class]]**.

<h3 id="withHeaders">withHeaders(struct headerStruct)</h3>

**Use it inside:** responder methods inside your Resource CFCs (e.g. get, put, post, delete - as well as head, options, etc).<br/>
**Parameters:**

* headerStruct (struct) - A structure whose keys are desired header names ("x-powered-by") and whose associated values are the values for the corresponding headers ("Taffy 1.1!").

This special method  _**requires**_ the use of either **[noData](#noData)** or **[representationOf](#representationOf)**. It adds custom headers to the return. Additional use of **[withStatus](#withStatus)** optional.

<h3 id="withStatus">withStatus(numeric statusCode)</h3>

**Use it inside:** responder methods inside your Resource CFCs (e.g. get, put, post, delete - as well as head, options, etc).<br/>
**Parameters:**

* statusCode (numeric) - the [HTTP Status Code](http://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html) to return to the consumer.

This special method  _**requires**_ the use of either **[noData](#noData)** or **[representationOf](#representationOf)**. It sets the HTTP Status Code of the return. Additional use of **[withHeaders](#withHeaders)** optional.

_If you do not specify a return status code, Taffy will always return status code 200 (OK) by default._