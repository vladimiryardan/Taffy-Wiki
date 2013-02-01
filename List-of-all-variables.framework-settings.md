Variables.framework was added in Taffy 1.2, and supports the following settings. These are their default values:

```javascript
variables.framework = {
	reloadKey = "reload",
	reloadPassword = "true",
	reloadOnEveryRequest = false,

	representationClass = "taffy.core.nativeJsonRepresentation",

	dashboardKey = "dashboard",
	disableDashboard = false,

	unhandledPaths = "/flex2gateway",
	allowCrossDomain = false,
	globalHeaders = structNew(),
	debugKey = "debug",

	useEtags = false,

	returnExceptionsAsJson = true,
	exceptionLogAdapter = "taffy.bonus.LogToEmail",
	exceptionLogAdapterConfig = {
		emailFrom = "api-error@yourdomain.com",
		emailTo = "you@yourdomain.com",
		emailSubj = "Exception Trapped in API",
		emailType = "html"
	},

	beanFactory = "",

	environments = {}
};
```

### reloadKey

**Available in:** Taffy 1.2+<br/>
**Type:** String<br/>
**Default:** "reload"<br/>
**Description:** Name of the url parameter that requests the framework to be reloaded. Used in combination with the reload password (see: **reloadPassword**), the framework will re-initialize itself. During re-initialization, all configuration settings are re-applied and all cached objects are cleared and reloaded. If the value of the key does not match the reload password, a reload will not be performed. This allows you to set a secret password to restrict control of reloading your API to trusted parties.

### reloadPassword

**Available in:** Taffy 1.2+<br/>
**Type:** String<br/>
**Default:** "true"<br/>
**Description:** Accepted value of the url parameter that requests the framework to be reloaded. Used in combination with the reload key (see: **reloadKey**), the framework will re-initialize itself. During re-initialization, all configuration settings are re-applied and all cached objects are cleared and reloaded. If the value of the key does not match the reload password, a reload will not be performed. This allows you to set a secret password to restrict control of reloading your API to trusted parties.

### reloadOnEveryRequest

**Available in:** Taffy 1.2+<br/>
**Type:** Boolean<br/>
**Default:** False<br/>
**Description:** Flag that indicates whether Taffy should reload cached values and configuration on every request. Useful in development; set to FALSE in production.

### representationClass

**Available in:** Taffy 1.2+<br/>
**Type:** String<br/>
**Default:** "taffy.core.nativeJsonRepresentation"<br/>
**Description:** The CFC dot-notation path, or bean name, of the [[representation class|Using-a-Custom-Representation-Class]] that your API will use to serialize returned data for the client.

### dashboardKey

**Available in:** Taffy 1.2+<br/>
**Deprecated in:** Taffy 1.3+<br/>
**Type:** String<br/>
**Default:** "dashboard"<br/>
**Description:** Name of the url parameter that displays the dashboard. The dashboard displays resources that your API is aware of, generates documentation about your API based on **hint** attributes, and contains a mock client to make testing your API easy.

### disableDashboard

**Available in:** Taffy 1.2+<br/>
**Type:** Boolean<br/>
**Default:** False<br/>
**Description:** Whether or not Taffy will allow the dashboard to be displayed. If set to true, the dashboard key is simply ignored. You may wish to disable the dashboard in production, depending on whether or not you want customers/clients to be able to see it.

### unhandledPaths

**Available in:** Taffy 1.2+<br/>
**Type:** String (Comma-delimited list)<br/>
**Default:** "/flex2gateway"<br/>
**Description:** Set a list of paths (usually subfolders of the API) that you do not want Taffy to interfere with. Unless listed here, Taffy takes over the request lifecycle and does not execute the requested ColdFusion template.

### allowCrossDomain

**Available in:** Taffy 1.2+<br/>
**Type:** Boolean<br/>
**Default:** False<br/>
**Description:** Whether or not to allow cross-domain access to your API.

Turning this on adds the following headers:
```cfm
<cfheader name="Access-Control-Allow-Origin" value="*" />
<cfheader name="Access-Control-Allow-Methods" value="#allowedVerbs#" />
<cfheader name="Access-Control-Allow-Headers" value="Content-Type" />
```

The allowed verbs, of course, are the ones allowed by the requested resource, as well as OPTIONS.

### globalHeaders

**Available in:** Taffy 1.2+<br/>
**Type:** Structure<br/>
**Default:** `{}`<br/>
**Description:** A structure where each key is the name of a header you want to return, such as "X-MY-HEADER" and the structure value is the header value.

Global headers are static. You set them on application initialization and they do not change. If you need dynamic headers, you can add them to each response at runtime using `withHeaders()`.

### debugKey

**Available in:** Taffy 1.2+<br/>
**Type:** String<br/>
**Default:** "debug"<br/>
**Description:** Name of the url parameter that enables CF Debug Output.

### useEtags

**Available in:** Taffy 1.3+<br/>
**Type:** Boolean<br/>
**Default:** False<br/>
**Description:** Enable the use of [HTTP ETags](http://en.wikipedia.org/wiki/HTTP_ETag) for caching purposes. Taffy will automatically handle both sending the server ETag value and detecting client supplied ETags (via the `If-None-Match` header) for you; simply turn this setting on.

_NOTE FOR RAILO USERS:_ While it will not cause errors, the underlying Java code used in this feature was improperly implemented prior to **Railo 4.0.?** and this could result in your result data being sent as if it were changed when it in fact has not. (I'm not sure which Railo point release will include the fix. The latest as of this writing is version 4.0.2, and does not include it.) _Adobe ColdFusion is unaffected._

### returnExceptionsAsJson

**Available in:** Taffy 1.2+<br/>
**Type:** Boolean<br/>
**Default:** true<br/>
**Description:** When an error occurs that is not otherwise handled, this option tells Taffy to attempt to format the error information as JSON and return that (regardless of the requested return format).

### exceptionLogAdapter

**Available in:** Taffy 1.2+<br/>
**Type:** String<br/>
**Default:** "taffy.bonus.LogToEmail"<br/>
**Description:** CFC dot-notation path to the exception logging adapter you want to use. Default adapter simply emails all exceptions. See [[Exception Logging Adapters]] for more details.

### exceptionLogAdapterConfig

**Available in:** Taffy 1.2+<br/>
**Type:** Any<br/>
**Default:** See values at top of this page<br/>
**Description:** Configuration that your chosen logging adapter requires. Can be any data type. See [[Exception Logging Adapters]] for more details.

### beanFactory

**Available in:** Taffy 1.2+<br/>
**Type:** Object Instance<br/>
**Default:** ""<br/>
**Description:** Already instantiated and cached (e.g. in Application scope) object instance of your external bean factory. Not required in order to use Taffy's built-in factory.

### environments

**Available in:** Taffy 1.3+<br/>
**Type:** Structure<br/>
**Default:** `{}`<br/>
**Description:** Environment-specific overrides to any framework settings. Applied after general `variables.framework` settings, _and after `configureTaffy()` has been called_. See [[Environment Specific Configuration]] for more details.


## configureTaffy() -> (Deprecated)

Prior to version 1.2 of Taffy (in version 1.1 and earlier) the recommended practice for setting up this configuration was through the use of a method named [[configureTaffy|Taffy1.1:-Index-of-API-Methods]] and individual setter methods for each of the settings.

configureTaffy and its related setters have all been officially deprecated as of version 1.2, and is scheduled to be removed no earlier than version 2.0. Please make sure your code uses `variables.framework` instead.

### Settings precedence when using variables.framework and configureTaffy()

As described above, `configureTaffy` is deprecated and will be going away in the future. For now it is still supported. Here's how precedence works if you find yourself using a mix of it and variables.framework:

1. Default values are set into memory
1. variables.framework is used to overwrite any default values that may be duplicated
1. if defined, configureTaffy is run

Because of this, values set by configureTaffy currently take precedence over values in variables.framework.