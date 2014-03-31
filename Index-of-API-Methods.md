This page is an alphabetical listing of all methods that Taffy exposes for you to use in your APIs, grouped by location that they are available. If a method is not included here, you should consider it un-documented, and as such it may change or be renamed at any time without notice. I will make a conscious effort not to break or rename any documented functions, though they may change with a major version if deemed an improvement to the framework.

* Application.cfc Methods
  * applicationStartEvent <em>(Obsoleted as of v2.0 see [[Migration-Guide|Migration-Guide#wiki-applicationstartevent-and-requeststartevent-were-removed]])</em>
  * getPath
  * getBasicAuthCredentials
  * getBeanFactory
  * newRepresentation
  * onTaffyRequest
  * requestStartEvent <em>(Obsoleted as of v2.0 see [[Migration-Guide|Migration-Guide#wiki-applicationstartevent-and-requeststartevent-were-removed]])</em>
  * Deprecated as of 1.2
     * <em>configureTaffy</em>
     * <em>enableCrossDomainAccess</em>
     * <em>enableDashboard</em>
     * <em>getGlobalHeaders</em>
     * <em>setBeanFactory</em>
     * <em>setDashboardKey</em>
     * <em>setDebugKey</em>
     * <em>setDefaultRepresentationClass</em>
     * <em>setGloablHeaders</em>
     * <em>setReloadKey</em>
     * <em>setReloadPassword</em>
     * <em>setUnhandledPaths</em>
  * Deprecated as of 1.1
     * <em>registerMimeType</em>
     * <em>setDefaultMime</em>
* Resource CFC Methods
  * noData
  * [[queryToArray|Index-of-API-Methods#queryToArray]]
  * [[queryToStruct|Index-of-API-Methods#queryToStruct]]
  * representationOf
  * saveLog
  * streamBinary
  * streamFile
  * streamImage
  * withHeaders
  * withMime
  * withStatus

## Application.cfc Methods

The following methods are available in your Application.cfc:

### applicationStartEvent()

<em>(Obsoleted as of v2.0 see [[Migration-Guide|Migration-Guide#wiki-applicationstartevent-and-requeststartevent-were-removed]])</em>

**Use it inside:** Application.cfc<br/>
**Parameters:** _(none)_

Since the framework takes over the **onApplicationStart** event of Application.cfc, it also exposes this methdod as a way for you to add logic to be executed when the event occurs. **applicationStartEvent** is called by **onApplicationStart** _before_ the framework is initialized. It is recommended that your Application.cfc **NOT implement onApplicationStart**, and should instead use **applicationStartEvent**.

### <em>configureTaffy()</em> (deprecated)

(deprecated as of 1.2; see [[variables.framework|List-of-all-variables.framework-settings]] instead)

**Use it inside:** Application.cfc<br/>
**Parameters:** _(none)_

This is a special method called by the framework during initialization to get your APIs configuration. You must implement this method in your Application.cfc to override any framework setting defaults. All configuration methods should be used inside your implementation of **configureTaffy**, if you intend to use them.

### <em>enableCrossDomainAccess(boolean enabled)</em> (deprecated)

(deprecated as of 1.2; see [[variables.framework|List-of-all-variables.framework-settings]] instead)

**Use it inside:** configureTaffy<br/>
**Parameters:**

* enabled (boolean) - whether or not to allow cross-domain access to your api

Turning this on adds the following headers:
```cfm
<cfheader name="Access-Control-Allow-Origin" value="*" />
<cfheader name="Access-Control-Allow-Methods" value="#allowedVerbs#" />
<cfheader name="Access-Control-Allow-Headers" value="Content-Type" />
```

The allowed verbs, of course, are the ones allowed by the requested resource, as well as OPTIONS.

### <em>enableDashboard(boolean enabled)</em> (deprecated)

(deprecated as of 1.2; see [[variables.framework|List-of-all-variables.framework-settings]] instead)

**Use it inside:** configureTaffy<br/>
**Parameters:**

* enabled (boolean) - Whether or not Taffy will allow the dashboard to be displayed. If set to false, the dashboard key is simply ignored. Default value is **TRUE**.

You should probably set this to FALSE in production to prevent snooping around.

### getPath()

**Use it inside:** Application.cfc<br/>
**Parameters:** _(none)_

This method is provided as an extension point. On Adobe ColdFusion ("ACF") 9, installed in standard entire-server mode, no change should be necessary. However, if ACF is installed on another JEE app server (i.e. Tomcat, Glassfish, etc), or on JRun but using an EAR/WAR setup, then you may need to override this method to make Taffy work on your server. See [[GetPath Setups]] for more information.

### getBasicAuthCredentials()

**Use it inside:** Application.cfc<br/>
**Parameters:** _(none)_

**Added in Taffy 1.3.** This method returns a structure with two keys: `username`, and `password`. When the client does not provide HTTP Basic Auth credentials, the username and password keys will be blank. When the client does provide them, the values will be available in these keys.

This method is only available inside your Application.cfc. If, for example, you need access to the username in a resource, you can use this method inside onTaffyRequest and add the username to the requestArguments structure.

### getBeanFactory()

**Use it inside:** Application.cfc<br/>
**Parameters:** _(none)_

Returns whatever bean factory you may have set into Taffy, if any.

### <em>getGlobalHeaders()</em> (deprecated)

(deprecated as of 1.2; see [[variables.framework|List-of-all-variables.framework-settings]] instead)

**Use it inside:** Application.cfc<br/>
**Parameters:** _(none)_

Returns the structure of global headers that you have set using `setGlobalHeaders()`, if any. If none, returns an empty structure.

### newRepresentation(string class)

**Use it inside:** onTaffyRequest<br/>
**Parameters:**

* class (string) - the dot-notation cfc path (or bean id if managing the class with a bean factory, including the built-in factory) for the representation class. Optional. Default value is the default representation class, `nativeJsonRepresentation`.

Use this method inside onTaffyRequest when you want to _abort_ the request with some specific message or data. Use the `setData` method on the returned representation class to put your data/message into it before returning it.

### onTaffyRequest(string verb, string cfc, struct requestArguments, string mimeExt, struct headers)

**Use it inside:** Application.cfc<br/>
**Parameters:**

* verb (string) - The HTTP request verb provided by the client
* cfc (string) - The CFC name (minus ".cfc") that would handle the request. (Bean Name, if using an external bean factory.)
* requestArguments (struct) - A structure containing all of the arguments of the request, including tokens from the URI as well as any query string parameters (defined after the ?, eg ?city=).
* mimeExt (string) - The mime extension (e.g. "json" - NOT the full mime type, e.g. "application/json")
* headers (struct) - A structure containing each header from the request, as sent by the client.

This method is optional, and allows you to inspect and potentially abort an API request in a way that adheres to the HTTP specification. If you choose not to override it (by implementing it in your Application.cfc), it will always return true, allowing the request to continue. If you implement it, you can check for things like an API key, or whether or not the customer has paid for your service, and return something other than the data that they are requesting.

If you do not return TRUE, allowing the request to continue as normal, then Taffy expects you to return a **[representation](/atuttle/Taffy/wiki/Using-a-Custom-Representation-Class)** (either the default class, or a custom one) that it should immediately return to the consumer, serialized to the appropriate format. If you simply want to return with a status code of 403 (which indicates "Not Allowed"), you could do this:

```cfs
return newRepresentation().noData().withStatus(403);
```

Alternately, you could return some data to indicate that they owe you money or something:

```cfs
return newRepresentation("taffy.core.nativeJsonRepresentation")
       .setData({error="Your account is past due. Please email accounts payable."})
       .withStatus(403);
```

The options here are limited only by your imagination.

You can add data to the **requestArguments** structure and this will be passed on to any resource that handles the request. Simply add a key to the structure:

In Application.cfc:

```cfs
function onTaffyRequest(verb, cfc, requestArguments, mimeExt, headers){
  arguments.requestArguments.myData = "myvalue";
}
```

In your resource:
```cfs
function get(myData){
  //arguments.myData = myValue
}
```

### requestStartEvent()

<em>(Obsoleted as of v2.0 see [[Migration-Guide|Migration-Guide#wiki-applicationstartevent-and-requeststartevent-were-removed]])</em>

**Use it inside:** Application.cfc<br/>
**Parameters:** _(none)_

Since the framework takes over the **onRequestStart** event of Application.cfc, it also exposes this methdod as a way for you to add logic to be executed when the event occurs. **requestStartEvent** is called by **onRequestStart** after the framework is re-initialized (if requested). Your Application.cfc **should NOT implement onRequestStart**, and should instead use **requestStartEvent**.

### <em>registerMimeType(string extension, string mimeType)</em> (deprecated)

**This method is deprecated. It will likely be removed in a future update. [Use metadata instead](/atuttle/Taffy/wiki/Configuration-via-Metadata).**

**Use it inside:** configureTaffy<br/>
**Parameters:**

* extension (string) - The url-extension that will invoke this mime type. (e.g. "xml" or "json")
* mimeType (string) - The mime type that should be set into the http headers when the corresponding extension is requested. (e.g. "application/xml" or "application/json")

Each mime type that your API is capable of returning needs to be registered with the framework, and the **configureTaffy** method is the most efficient place to do so.

Because the framework implements JSON by default, the JSON ("application/json") mime type is already registered, and you do not have to re-register it. (You can if you like, though. It won't hurt anything.)

### <em>setBeanFactory(object beanFactory, [string beanList])</em> (deprecated)

(deprecated as of 1.2; see [[variables.framework|List-of-all-variables.framework-settings]] instead)

**Use it inside:** configureTaffy<br/>
**Parameters:**

* beanFactory (object) - already instantiated and cached (e.g. in Application scope) object instance of your external bean factory.
* beanList (string) - a comma-delimited list of bean Id's that the bean factory is aware of. Optional, and only necessary if the beanFactory does not implement `getBeanList()` _and is not ColdSpring_. Tight ColdSpring integration is baked in.

While Taffy includes a very basic Bean Factory (used by default) and also has fairly tight integration with ColdSpring, other bean factories could be added. If your bean factory implements `getBeanList()`, then Taffy will use that to get a list of beans that the factory knows about. However, if it does not, then Taffy will allow you to provide the bean list as a string in the optional second argument.

### <em>setDashboardKey(string keyName)</em> (deprecated)

(deprecated as of 1.2; see [[variables.framework|List-of-all-variables.framework-settings]] instead)

**Use it inside:** configureTaffy<br/>
**Parameters:**

* keyName (string) - Name of the url parameter that displays the dashboard. Default value is "dashboard".

The dashboard displays resources that your API is aware of, generates documentation about your API based on **hint** attributes, and contains a mock client to make testing your API easy.

### <em>setDebugKey(string keyName)</em> (deprecated)

(deprecated as of 1.2; see [[variables.framework|List-of-all-variables.framework-settings]] instead)

**Use it inside:** configureTaffy<br/>
**Parameters:**

* keyName (string) - Name of the url parameter that enables CF Debug Output. (e.g. "debug")

Sometimes it's useful to see ColdFusion's debug output in the results of your API requests during testing. For that reason, you can include this query string parameter to enable debugging on a per-request basis.

_If you do not change it, the default value is, "debug"._

### <em>setDefaultMime(string DefaultMimeType)</em> (deprecated)

**This method is deprecated. It will likely be removed in a future update. [Use metadata instead](/atuttle/Taffy/wiki/Configuration-via-Metadata).**

**Use it inside:** configureTaffy<br/>
**Parameters:**

* DefaultMimeType (string) - The mime type that should be returned when none is specified by the consumer. (e.g. "json")

It is important to note the difference between the mime type and extension. Here, the extension is expected because of its brevity. While the mime type to be returned is "application/json", you should pass the string "json" to the function. If, for example, you registered the mime type of "xml" as "application/xml" (using **registerMimeType**), and you wanted XML to be the default format, you would pass the string "xml" to the **setDefaultMime** function; and it could be specified on the url by appending ".xml" to the URI, before any query string parameters, like so:

`/artists.xml?city=Philadelphia`

_If not implemented, the framework default mime type is JSON ("application/json")._

### <em>setDefaultRepresentationClass(string customClassDotPath)</em> (deprecated)

(deprecated as of 1.2; see [[variables.framework|List-of-all-variables.framework-settings]] instead)

**Use it inside:** configureTaffy<br/>
**Parameters:**

* customClassDotPath (string) - Dot-notation path to the CFC to be used by default. Alternately, you can pass in a beanId and the class will be requested from the bean factory. This works with both the internal bean factory (your `/resources` folder) and if you are using an external bean factory.

When you change the default Representation Class, all responses will use your custom default class, unless you specifically override that request, using the (optional) second parameter to the **representationOf** function.

### <em>setGlobalHeaders(struct headers)</em> (deprecated)

(deprecated as of 1.2; see [[variables.framework|List-of-all-variables.framework-settings]] instead)

**Use it inside:** configureTaffy<br/>
**Parameters:**

* headers (struct) - A structure where each key is the name of a header you want to return, such as "X-MY-HEADER" and the structure value is the header value

Global headers are static. You set them on application initialization and they do not change. If you need dynamic headers, you can add them to each response at runtime using `withHeaders()`.

### <em>setReloadKey(string keyName)</em> (deprecated)

(deprecated as of 1.2; see [[variables.framework|List-of-all-variables.framework-settings]] instead)

**Use it inside:** configureTaffy<br/>
**Parameters:**

* keyName (string) - Name of the url parameter that requests the framework to be reloaded. (e.g. "reload")

Used in combination with the reload password (see: **setReloadPassword**), the framework will re-initialize itself. During re-initialization, all configuration settings are re-applied and all cached objects are cleared and reloaded. If the value of the key does not match the reload password, a reload will not be performed. This allows you to set a secret password to restrict control of reloading your API to trusted parties.

_If you do not change it, the default value is "reload"._

### <em>setReloadPassword(string Password)</em> (deprecated)

(deprecated as of 1.2; see [[variables.framework|List-of-all-variables.framework-settings]] instead)

**Use it inside:** configureTaffy<br/>
**Parameters:**

* keyName (string) - Accepted value of the url parameter that requests the framework to be reloaded. (e.g. "true")

Used in combination with the reload key (see: **setReloadKey**), the framework will re-initialize itself. During re-initialization, all configuration settings are re-applied and all cached objects are cleared and reloaded. If the value of the key does not match the reload password, a reload will not be performed. This allows you to set a secret password to restrict control of reloading your API to trusted parties.

_If you do not change it, the default value is, "true"._

### <em>setUnhandledPaths(string unhandledPaths)</em> (deprecated)

(deprecated as of 1.2; see [[variables.framework|List-of-all-variables.framework-settings]] instead)

**Use it inside:** configureTaffy<br/>
**Parameters:**

* unhandledPaths (string) - new list of unhandled paths, comma-delimited (commas may not be part of any list item)

Use this method to set a list of paths (usually subfolders of the api) that you do not want Taffy to interfere with. Unless listed here, Taffy takes over the request lifecycle and does not execute the requested ColdFusion template.

## Resource CFC Methods:

Resource CFCs extend `taffy.core.resource`.  The following methods are available inside each of your Resource CFCs:

### noData()

**Use it inside:** responder methods inside your Resource CFCs (e.g. get, put, post, delete - as well as head, options, etc).<br/>
**Parameters:** _(none)_

This method allows you to specify that there is no data to be returned for the current request. Generally, you would use it in conjunction with the **withStatus** method to set a specific return status for the request. For example, if the requested resource doesn't exist, you could return a 404 error like so:

```cfs
return noData().withStatus(404);
```

### queryToArray(query data)

**Use it inside:** responder methods inside your Resource CFCs (e.g. get, put, post, delete - as well as head, options, etc).<br/>
**Parameters:**

* data (query) - The query object to be transformed

This method transforms a ColdFusion query object into an array of structures. It was added because ColdFusion's serializeJSON functionality uses an ...eccentric... format for queries. queryToArray returns the format most people expect: a vanilla array of structures with named keys. To be fair the ACF serialization format uses less data as long as there is more than 1 row in the query, but it doesn't matter that you do a better job if nobody understands your output. _queryToArray also preserves query column name case, which serializeJSON does not._

```cfs
return representationOf(queryToArray(data)).withStatus(200);
```

### queryToStruct(query data)

**Use it inside:** responder methods inside your Resource CFCs (e.g. get, put, post, delete - as well as head, options, etc).<br/>
**Parameters:**

* data (query) - The query object to be transformed

This method transforms a ColdFusion query object with a single record into a structure. This achieves the same result as queryToArray(query data)[0]

```cfs
return representationOf(queryToStruct(data)).withStatus(200);
```

### representationOf(any data [, string customRepresentationClass])

**Use it inside:** responder methods inside your Resource CFCs (e.g. get, put, post, delete - as well as head, options, etc).<br/>
**Parameters:**

* data (any) - The data to return to the consumer.
* _optional_ customRepresentationClass (string) - Dot-notation path to a custom CFC, or bean name, that will serialize your results. Defaults to included generic serializer which supports JSON.

Data can be of any type, including complex data types like queries, structures, and arrays, as long as the serializer knows how to serialize them. For more information on using a custom representation class, **see [[Using a custom Representation Class]]**.

### saveLog(struct exception)

**Use it inside:** anywhere inside a Resource CFC to log data using your [configured logging adapter](https://github.com/atuttle/Taffy/wiki/Exception-Logging-Adapters).<br/>
**Parameters:**

* exception (struct) - traditionally a CF exception object, but any struct may be passed.

What you pass to this method is simply handed off to the logging adapter. You may use one of the included adapters (LogToEmail, LogToBuglogHQ, or LogToHoth), or a custom logging adapter. If you write a custom logging adapter, it should implement the `taffy.bonus.ILogAdapter` interface.

If you don't configure a logging adapter, the default is LogToEmail, but the default `from` and `to` email addresses are not useful. See [Exception Log Adapters](https://github.com/atuttle/Taffy/wiki/Exception-Logging-Adapters) for more information on configuring logging adapters.

### streamBinary(any binaryData [, string customRepresentationClass])

**Use it inside:** responder methods inside your Resource CFCs (e.g. get, put, post, delete - as well as head, options, etc).<br/>
**Parameters:**

* binaryData (any) - the binary data to be streamed back to the consumer
* _optional_ customRepresentationClass (string) - Dot-notation path to a custom CFC, or bean name, that will serialize your results. Defaults to included generic serializer which supports JSON.

Use this method in place of `representationOf()` to return a stream of binary data. Useful for streaming things like dynamically generated PDFs.

### streamFile(string fileName [, string customRepresentationClass])

**Use it inside:** responder methods inside your Resource CFCs (e.g. get, put, post, delete - as well as head, options, etc).<br/>
**Parameters:**

* fileName (string) - fully qualified file path (eg c:\tmp\files.zip)
* _optional_ customRepresentationClass (string) - Dot-notation path to a custom CFC, or bean name, that will serialize your results. Defaults to included generic serializer which supports JSON.

Use this method in place of `representationOf()` to stream a file from disk (or VFS).

### streamImage

**Use it inside:** responder methods inside your Resource CFCs (e.g. get, put, post, delete - as well as head, options, etc).<br/>
**Parameters:**

* binaryData (any) - the binary, base64 encoded data of the image, to be streamed back to the consumer
* _optional_ customRepresentationClass (string) - Dot-notation path to a custom CFC, or bean name, that will serialize your results. Defaults to included generic serializer which supports JSON.

Use this method in place of `representationOf()` to stream an image from disk (or VFS).

### withHeaders(struct headerStruct)

**Use it inside:** responder methods inside your Resource CFCs (e.g. get, put, post, delete - as well as head, options, etc).<br/>
**Parameters:**

* headerStruct (struct) - A structure whose keys are desired header names ("x-powered-by") and whose associated values are the values for the corresponding headers ("Taffy 1.1!").

This special method  _**requires**_ the use of either **noData** or **representationOf**. It adds custom headers to the return. Additional use of **withStatus** optional.

Ex: `return representationOf(myData).withHeaders({"X-POWERED-BY"="Taffy 1.1!"});`

### withMime(string mime)

**Use it inside:** responder methods inside your Resource CFCs (e.g. get, put, post, delete - as well as head, options, etc).<br/>
**Parameters:**

* mime (string) - mime type (eg. "application/pdf") to be returned with the streamed file data

This special method _**requires**_ the use of either **streamFile** or **streamBinary**. It overrides the default mime type header for the return.

Ex: `return streamFile('kittens/cuteness.pdf').withMime('application/pdf');`

### withStatus(numeric statusCode)

**Use it inside:** responder methods inside your Resource CFCs (e.g. get, put, post, delete - as well as head, options, etc).<br/>
**Parameters:**

* statusCode (numeric) - the [HTTP Status Code](http://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html) to return to the consumer.

This special method  _**requires**_ the use of either **noData** or **representationOf**. It sets the HTTP Status Code of the return. Additional use of **withHeaders** optional.

_If you do not specify a return status code, Taffy will always return status code 200 (OK) by default._

Ex: `return noData().withStatus(404);`