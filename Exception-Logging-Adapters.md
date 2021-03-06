As of Taffy 1.2, additional functionality has been added to abstract logging and reporting of exceptions that occur within your Taffy APIs. This is done using adapters that provide a standardized API.

Taffy comes with a number of adapters:

1. taffy.bonus.LogToEmail: Sends an email on each exception  (default prior to Taffy 2.2.2)
1. taffy.bonus.LogToBugLogHQ: Logs exceptions to [BugLogHQ](https://github.com/oarevalo/BugLogHQ)
1. taffy.bonus.LogToHoth: Logs exceptions to [Hoth](https://github.com/aarongreenlee/Hoth)
1. taffy.bonus.LogToDevNull: Doesn't log anything anywhere. Exceptions still returned as results for debugging purposes. (Included as of Taffy 2.1.0)
1. taffy.bonus.LogToScreen: `<cfdump>`'s the exception into the response. Maybe you don't like looking at exceptions as JSON? (Included as of Taffy 2.1.0, default as of Taffy 2.2.2)
You specify which adapter you want Taffy to use in `variables.framework.exceptionLogAdapter`. Each of these three adapters requires different configuration, which you provide via `variables.framework.exceptionLogAdapterConfig`.
1. taffy.bonus.LogToBugsnag: Logs exceptions to [Bugsnag](https://bugsnag.com)

Note that Taffy's default is to use LogToEmail; but the default configuration does not have your email address. You should either specify proper values for the email configuration (see below) or switch to a different adapter.

## LogToEmail

Adapter Path: taffy.bonus.LogToEmail<br/>
Configuration Options: (structure)

```javascript
variables.framework.exceptionLogAdapterConfig = {
	emailFrom = "api-error@yourdomain.com",
	emailTo = "you@yourdomain.com",
	emailSubj = "Exception Trapped in API",
	emailType = "html"
};
```

Hopefully the first 3 are completely self explanatory. The last one may bear a tiny bit of explanation. You can choose either "html" or "text". If you choose "html" (the default), then Taffy will include an HTML CFDump of the exception in the email. If you choose "text" (only available on CF8+) then Taffy will include a text-based CFDump. Plain text results in significantly smaller emails, but at the cost of a little bit of readability.

## LogToHoth

Adapter Path: taffy.bonus.LogToHoth<br/>
Configuration Options: (object) There are numerous settings for Hoth, but I recommend that you look at [HothConfig.cfc](https://github.com/atuttle/Taffy/blob/develop/examples/api_Hoth/resources/HothConfig.cfc) in `examples/api_Hoth/resources`. From there it should be pretty self explanatory for anyone that's used Hoth before.

A typical Application.cfc configuration to use Hoth might resemble the following:

```javascript
variables.framework.exceptionLogAdapter = "taffy.bonus.LogToHoth";
variables.framework.exceptionLogAdapterConfig = "taffy.examples.api_hoth.resources.HothConfig";
```

Note that the config is _also_ a CFC dot-notation path. The adapter automatically instantiates your config and provides it to Hoth as needed.



There is an example provided in: `examples/api_Hoth`.

## LogToBugLogHQ

Adapter Path: taffy.bonus.LogToBugLogHQ<br/>
Configuration Options: (structure)

```javascript
variables.framework.exceptionLogAdapter = "taffy.bonus.LogToBugLogHQ";

variables.framework.exceptionLogAdapterConfig = StructNew();
variables.framework.exceptionLogAdapterConfig.bugLogListener = "bugLog.listeners.bugLogListenerWS";
variables.framework.exceptionLogAdapterConfig.bugEmailRecipients = "you@yourdomain.com";
variables.framework.exceptionLogAdapterConfig.bugEmailSender = "errors@yourdomain.com";
variables.framework.exceptionLogAdapterConfig.hostname = "Taffy_DEV_Examples";
variables.framework.exceptionLogAdapterConfig.apikey = "";
```

**bugLogListener:** Currently, Taffy requires that the bugLogService client be available at "buglog.client.bugLogService" -- but that does not require you to be running BugLogHQ on the same server. You may still use the REST/SOAP listeners to send exceptions to a remote server.

**bugEmailRecipients:** If for some reason the BugLogService client can't connect to your BugLogHQ instance, it will send you an email instead. Here you can specify a list of recipients of that email.

**bugEmailSender:** Whom the email should be sent from.

**hostname:** The hostname that should appear in the bug reporting. It should be specific to the environment, so that you can look at the report and know exactly where to find the problem code. Specify dev/staging/production/etc, as well as any other necessary identifying information.

**apikey:** If your BugLogHQ instance requires an API Key to submit reports, supply it here.

There is an example provided in: `examples/api_BugLogHQ`.

## LogToBugsnag

Adapter Path: taffy.bonus.LogToBugsnag<br/>
Configuration Options: (structure)

```javascript
variables.framework.exceptionLogAdapterConfig = {
	apiKey = "c9d60ae4c7e70c4b6c4ebd3e8056d2b8",
	appVersion = "1.1.3",
	releaseStage = "production"
};
```

**apiKey:** The API Key associated with the project.

**appVersion:** The version number of the application which generated the error.

**releaseStage:** The release stage that this error occurred in (e.g "development", "staging" or "production")

# Custom Log Adapter

You may write your own custom log adapter. It should implement the `taffy.bonus.ILogAdapter` interface.

Note the way that configuration is passed to the logging adapter. Both the adapter path and a configuration variable are defined in Application.cfc and the adapter is only instantiated and configured (via `init()`) when needed.

If you create a custom log adapter, you should clearly define the expected structure and supported values for the configuration data.