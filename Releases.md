## Version 1.3
**Released:** April 12, 2013

* Bugs Fixed:
  * [#105](https://github.com/atuttle/Taffy/issues/105) - Static URIs that would match dynamic URIs are now allowed (`/user/logout` would previously conflict with `/user/{userId}`). Pay special attention to [URI Matching Order](https://github.com/atuttle/Taffy/wiki/Configuration-via-Metadata#uri-matching-order).
  * [#120](https://github.com/atuttle/Taffy/issues/120) - Added support for `*/*` Accept header value. _Thanks to Brian Quackenbush_ for the patch!
  * [#124](https://github.com/atuttle/Taffy/issues/124) - Fixed a Railo-specific bug blocking resources from being loaded when the resources folder isn't in the web root. _Thanks to Jean-Bernard van Zuylen for the initial bug report and the [pull request!](https://github.com/atuttle/Taffy/pull/133)!_
  * [#129](https://github.com/atuttle/Taffy/issues/129) - Fixed a bug that MANY people have asked about recently: when you don't supply _any_ form of requested return format (via header or URL "extension"), the dreaded "your default mime type is not implemented" error was returned. Jean-Bernard van Zuylen provided an [epic detailed bug report](https://github.com/atuttle/Taffy/issues/129), as well as [the pull request that ultimately fixed it](https://github.com/atuttle/Taffy/pull/132). A regular open source hero!
  * [#130](https://github.com/atuttle/Taffy/issues/130) - Fixed a regression from 1.2 in changes to the dashboard to use the new endpoint url param. _This one was also reported and fixed via pull request by Jean-Bernard van Zuylen._
* New Features:
  * [#91](https://github.com/atuttle/Taffy/issues/91) - A [message](https://www.evernote.com/shard/s240/sh/6b166322-d8a8-4209-8de1-7348abd8baca/3b4072548cd291ded70ae60f1d4d5583/res/1539c94d-d644-48cc-883a-cfb80e37c4e5/skitch.png) is now displayed if Taffy can't find any resources.
  * [#99](https://github.com/atuttle/Taffy/issues/99) - Show dashboard without the `?dashboard` query param (just browse to the root of your API). See deprecations, below.
  * [#102](https://github.com/atuttle/Taffy/issues/102) - Added support for DI/1 bean factory. See [this example](https://github.com/atuttle/Taffy/blob/1.3-dev/examples/api_DI1/Application.cfc) for a sample implementation.
  * [#103](https://github.com/atuttle/Taffy/issues/103) - Added [environment-based configuration](https://github.com/atuttle/Taffy/wiki/Environment-Specific-Configuration).
  * [#108](https://github.com/atuttle/Taffy/issues/108) - Added helper method `getBasicAuthCredentials()` to api.cfc (so you can [use it in your Application.cfc](https://github.com/atuttle/Taffy/wiki/Authentication-and-Security)). It returns a structure with keys `username` and `password`, and if _NO_ basic auth credentials have been included in the request then both values will be blank.
  * [#109](https://github.com/atuttle/Taffy/issues/109) - Added helper method `saveLog()` to resource classes, which delegates to your [configured exception logger](https://github.com/atuttle/Taffy/wiki/Exception-Logging-Adapters). You may now use: `saveLog(cfcatch)` from inside a resource.
  * [#115](https://github.com/atuttle/Taffy/issues/115) - You can now [use properties instead of setters](https://github.com/atuttle/Taffy/wiki/So-you-want-to:-Use-Taffy's-built-in-Dependency-Injection-to-resolve-dependencies-of-your-resources) to have Taffy autowire dependencies. (Setters are not deprecated; this is just an additional option.)
  * [#117](https://github.com/atuttle/Taffy/pull/117) - Added support for [endpointURLParam](https://github.com/atuttle/Taffy/wiki/List-of-all-variables.framework-settings). _Thanks [Marco Betschart](https://github.com/marbetschar)_.
  * [#122](https://github.com/atuttle/Taffy/issues/122) - Added support for [ETag based caching](https://github.com/atuttle/Taffy/wiki/List-of-all-variables.framework-settings)
  * [#128](https://github.com/atuttle/Taffy/issues/128) - Now support **Access-Control-Allow-Headers** to list allowable headers for Cross-Domain requests. _Thanks to [Marco Betschart](https://github.com/marbetschar) for the bug report and the patch!_

* Deprecations:
  * Using `?dashboard` to display the dashboard is now deprecated in favor of simply browsing to the root of your API, with or without /index.cfm in the url. E.g. http://api.acme.com/ or http://api.acme.com/index.cfm instead of the old http://api.acme.com/?dashboard

## Version 1.2
**Released:** December 27, 2012

* Bugs fixed:
  * [#89](https://github.com/atuttle/Taffy/issues/89) - File Uploads did not work on Railo
  * [#93](https://github.com/atuttle/Taffy/issues/93) - Tokens where the expected value could contain a period character caused issues with URL format specification.
  * [#97](https://github.com/atuttle/Taffy/issues/97) - Method PUT was not allowed by Access-Control-Allow-Methods header.
* New Features:
  * [#48](https://github.com/atuttle/Taffy/issues/48) - Organizing resources into subfolders of `/resources` is [[now supported|Organizing your resources into subfolders]].
  * [#57](https://github.com/atuttle/Taffy/issues/57) - Added the ability to pass data from onTaffyRequest to resources. Add keys to the `requestArguments` argument, and they will be passed on to the resource, by name, just like URI tokens and query string parameters.
  * [#60](https://github.com/atuttle/Taffy/issues/60) - Custom Token Regular Expressions. For examples of using custom regular expressions for your tokens, see [[Custom Token Regular Expressions]].
  * [#61](https://github.com/atuttle/Taffy/issues/61) - ALLOW header is returned for every request.
  * [#73](https://github.com/atuttle/Taffy/issues/73) - Integration with [Hoth](https://github.com/aarongreenlee/Hoth) and [BugLogHQ](https://github.com/oarevalo/BugLogHQ) for exception tracking and reporting. [[Details|Exception Logging Adapters]]
  * [#90](https://github.com/atuttle/Taffy/issues/90) - Configuration via "variables.framework", ala FW/1. See also: [[List of all variables.framework settings]].
  * [#92](https://github.com/atuttle/Taffy/issues/92) - Format via URI now takes precedence over format via header. ([[Reasoning|So-you-want-to:-Support-returning-multiple-formats]])
  * [#94](https://github.com/atuttle/Taffy/issues/94) - A new setting was added, allowing you to reload on every request; useful in development.
  * [#106](https://github.com/atuttle/Taffy/issues/106) - Made cross-domain support more robust. Now supplies Allow-Origin, Allow-Methods, and Allow-Headers headers.
  * [#111](https://github.com/atuttle/Taffy/issues/111) - Changed all references of "defaultRepresentationClass" to simply "representationClass", as we've moved away from supporting multiple classes in all relevant cases.
* Deprecations:
  * It has been proposed to deprecate the use of ".format" (e.g. ".json") in the URI to specify requested return format because of the difficulty it was causing with the old URI parser. The Parser Rewrite for [#93](https://github.com/atuttle/Taffy/issues/93) resolved this issue, so any informal deprecations are no longer necessary. Use .format to your hearts content.
  * Use of ConfigureTaffy and individual setter methods to specify configuration settings is now deprecated in favor of [[variables.framework|List-of-all-variables.framework-settings]].

## Version 1.1
**Released:** June 27, 2012

* **New Platforms:**
  * Now fully tested and supported on Railo 3.2+!
* New features:
  * Extensive test suite added - helpful for contributors to know they haven't broken anything
    * Update test suite to use Tags instead of script so it can be run on any CFML engine (What good are tests if you can't use them to test?)
  * Added support for HTTP "HEAD" verb
  * Added ability to add custom headers to a response
  * Moved mime-type declaration ("this api supports json,xml,etc") into representation class metadata, instead of api-level configuration.
  * Added a set of Eclipse/CFBuilder snippets to help speed Taffy development (it wasn't fast enough already?!)
  * Added ability to manage representation class with internal/external bean factory, as well as resolving dependencies using the bean factory.
  * Added several more example implementations to the examples folder
  * Added `getBeanFactory()` method to compliment `setBeanFactory()`.
  * [\#20](https://github.com/atuttle/taffy/issues/20) - Added setting for cross-domain resource sharing
  * [\#21](https://github.com/atuttle/taffy/issues/21) - Added global custom headers
  * [\#28](https://github.com/atuttle/taffy/issues/28) - Bean factory now throws an exception when requesting a non-existant bean - Thanks to [Brian Panulla](https://github.com/bpanulla)
  * [\#29](https://github.com/atuttle/taffy/issues/29) - HTTP Method Tunneling  - Thanks to [Brian Panulla](https://github.com/bpanulla)
  * [\#46](https://github.com/atuttle/Taffy/pull/46) - Added support for ColdSpring AOP on Taffy resources. - Thanks to [mgersting](https://github.com/mgersting)
  * Added index page to examples folder for people who may not have directory indexing on, or who are running a platform without it (eg. JRun) - Thanks to [Barney Boisvert](http://www.barneyb.com/barneyblog/)
  * Added example of rate limiting using onTaffyRequest
  * Support for direct streaming of binary results, such as image streaming, ala [PlaceKitten](http://www.placekitten.com), or a generated PDF (examples included).
  * Added notion of "unhandled paths" (similar to FW/1), where you can specify a set of subfolders inside your API that Taffy will not take over the request lifecycle.
  * Added `streamBinary()`, `streamFile()`, and `streamImage()` methods for streaming in memory files, files on disk, and images, respectively; as well as `withMime()` method for specifying the mime type of the streamed data.
* Bugs fixed:
  * [\#19](https://github.com/atuttle/taffy/issues/19) - JSON input data not properly detected & deserialized into arguments
  * [\#23](https://github.com/atuttle/taffy/issues/24) - Various issues with ColdSpring integration - Thanks to [Brian Panulla](https://github.com/bpanulla)
  * [\#31](https://github.com/atuttle/taffy/issues/31) - Error failsafe to display general exceptions in an api-friendly manner.
  * [\#32](https://github.com/atuttle/taffy/issues/32) - Issues with periods in token values (such as email addresses or IP addresses).
  * [\#33](https://github.com/atuttle/taffy/issues/33) - Error when attempting to redirect from api root to dashboard on internal web server (JRun)
  * [\#34](https://github.com/atuttle/taffy/issues/34) - Exception in error handling code for duplicate URI patterns on CF8 and earlier.
  * [\#38](https://github.com/atuttle/Taffy/pull/38) - Fixed handling of request body for some verbs
  * [\#49](https://github.com/atuttle/Taffy/pull/49) - Fix for parsing query strings with key-value pairs missing a value (a=1&b=&c=3) - Thanks to [Greg Moser](https://github.com/gregmoser)
  * [\#50](https://github.com/atuttle/Taffy/pull/50) Added `WEB-INF` to git ignore file, for J2EE/Railo users. - Thanks to [Dave Long](https://github.com/davidlong03)
  * [\#55](https://github.com/atuttle/Taffy/issues/55) - Support Apache request proxying by stripping the context root from the resources path if it's non-blank - Thanks to [Will Coleda](https://github.com/coke)
  * Fix errors on startup if Application context is shared with another app & Taffy doesn't get properly initialized.
* Deprecated features
  * Both `registerMimeType()` and `setDefaultMime()` are deprecated as of version 1.1; slated to be removed in 2.0. Use [[Configuration via Metadata]] instead.


## Version 1.1-rc1
**Published:**  October 27, 2011

## Version 1.0
**Released:** August 23, 2010 - [Announcement](http://fusiongrokker.com/post/taffy-a-restful-framework-for-coldfusion)