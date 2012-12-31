New in Taffy 1.3, you can now provide environment-specific configuration in a manner very similar to FW/1 -- in fact some of the code has even been borrowed from FW/1. Thanks, Sean!

We use a simplified approach.

## variables.framework additions

First, add a new key named `environments` to `variables.framework`. It should be a structure, and its key names should be your potential environment names. Each of these keys in the environments structure should also contain a structure with any settings you wish to override on this specific environment.

For example:

```cfs
variables.framework = {
	disableDashboard = false,
	environments = {
		production = {
			disableDashboard = true
		}
	}
};
```

In this example, the dashboard is disabled in production. But how does Taffy know from which environment key to get configuration overrides?

## getEnvironment()

Also new in Taffy 1.3, the `getEnvironment` method is available for you to override in your Application.cfc; and this is how Taffy determines which environment key to use (if any). By default this function returns an empty string. The intention is that you should override it, using whatever conditions you deem appropriate to return the applicable environment name as a string, such as `"production"`, `"development"`, or `"staging"`. Common approaches are to look at available environment variables such as `cgi.server_name`, or the server's hostname. If you so choose, you could even just hard-code a simple response:

```cfs
function getEnvironment() {
	return "production";
}
```

### getHostname()

Since getting the server's hostname could be useful, we also provide this helper function to return the current machine's hostname. For example:

```cfs
function getEnvironment() {
	if (getHostname() contains "prod") {
		return "production";
	} else {
		return "development";
	}
}
```