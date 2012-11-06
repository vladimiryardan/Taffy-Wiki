Variables.framework supports the following settings, and these are their default values:

```javascript
variables.framework = {
	debugKey = "debug",
	reloadKey = "reload",
	reloadPassword = "true",
	reloadOnEveryRequest = false,
	defaultRepresentationClass = "taffy.core.nativeJsonRepresentation",
	dashboardKey = "dashboard",
	disableDashboard = false,
	unhandledPaths = "/flex2gateway",
	allowCrossDomain = false,
	globalHeaders = structNew(),
	beanFactory = ""
};
```

### debugKey

Type: String<br/>
Default: "debug"<br/>
Description: Name of the url parameter that enables CF Debug Output.

### reloadKey

Type: String<br/>
Default: "reload"<br/>
Description: Name of the url parameter that requests the framework to be reloaded. Used in combination with the reload password (see: **reloadPassword**), the framework will re-initialize itself. During re-initialization, all configuration settings are re-applied and all cached objects are cleared and reloaded. If the value of the key does not match the reload password, a reload will not be performed. This allows you to set a secret password to restrict control of reloading your API to trusted parties.

### reloadPassword

Type: String<br/>
Default: "true"<br/>
Description: Accepted value of the url parameter that requests the framework to be reloaded. Used in combination with the reload key (see: **reloadKey**), the framework will re-initialize itself. During re-initialization, all configuration settings are re-applied and all cached objects are cleared and reloaded. If the value of the key does not match the reload password, a reload will not be performed. This allows you to set a secret password to restrict control of reloading your API to trusted parties.

### reloadOnEveryRequest

Type: Boolean<br/>
Default: False<br/>
Description: Flag that indicates whether Taffy should reload cached values and configuration on every request. Useful in development; set to FALSE in production.

### defaultRepresentationClass

Type: String<br/>
Default: "taffy.core.nativeJsonRepresentation"<br/>
Description: The CFC dot-notation path, or bean name, of the [[representation class|Using-a-Custom-Representation-Class]] that your API will use to serialize returned data for the client.

### dashboardKey

Type: String<br/>
Default: "dashboard"<br/>
Description: Name of the url parameter that displays the dashboard. The dashboard displays resources that your API is aware of, generates documentation about your API based on **hint** attributes, and contains a mock client to make testing your API easy.

### disableDashboard

Type: Boolean<br/>
Default: False<br/>
Description: Whether or not Taffy will allow the dashboard to be displayed. If set to true, the dashboard key is simply ignored. You may wish to disable the dashboard in production, depending on whether or not you want customers/clients to be able to see it.

### unhandledPaths

Type: String (Comma-delimited list)<br/>
Default: "/flex2gateway"<br/>
Description: Set a list of paths (usually subfolders of the API) that you do not want Taffy to interfere with. Unless listed here, Taffy takes over the request lifecycle and does not execute the requested ColdFusion template.

### allowCrossDomain

Type: Boolean<br/>
Default: False<br/>
Description: Whether or not to allow cross-domain access to your API.

Turning this on adds the following headers:
```cfm
<cfheader name="Access-Control-Allow-Origin" value="*" />
<cfheader name="Access-Control-Allow-Methods" value="#allowedVerbs#" />
<cfheader name="Access-Control-Allow-Headers" value="Content-Type" />
```

The allowed verbs, of course, are the ones allowed by the requested resource, as well as OPTIONS.

### globalHeaders

Type: Structure<br/>
Default: `{}`<br/>
Description: A structure where each key is the name of a header you want to return, such as "X-MY-HEADER" and the structure value is the header value.

Global headers are static. You set them on application initialization and they do not change. If you need dynamic headers, you can add them to each response at runtime using `withHeaders()`.

### beanFactory

Type: Object Instance<br/>
Default: ""<br/>
Description: Already instantiated and cached (e.g. in Application scope) object instance of your external bean factory. Not required in order to use Taffy's built-in factory.

## ConfigureTaffy -- Deprecated

Prior to version 1.2 of Taffy (in version 1.1 and earlier) the recommended practice for setting up this configuration was through the use of a method named [[configureTaffy|Taffy1.1:-Index-of-API-Methods]] and individual setter methods for each of the settings.

configureTaffy and its related setters have all been officially deprecated as of version 1.2, and is scheduled to be removed no earlier than version 2.0. Please make sure your code uses `variables.framework` instead.

### Settings precedence when using variables.framework and configureTaffy()

1. Default values are set into memory
1. variables.framework is used to overwrite any default values that may be duplicated
1. if defined, configureTaffy is run

Because of this, values set by configureTaffy currently take precedence over values in variables.framework.