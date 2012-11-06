## Version 1.2
**Released:** TBD

* Bug fixes:
  * File Uploads did not work on Railo

\([BUG89](https://github.com/atuttle/Taffy/issues/89)\) Fixed uploads on Railo.


## Bug Fix: Tokens where the expected value could contain a period character caused issues with URL format specification

\([BUG93](https://github.com/atuttle/Taffy/issues/93)\) For example, if you used a token for `{email}` in your resource URI, and an expected value was along the lines of "bob@gmail.com", the ".com" would confuse the URI parser and cause issues detecting the URL format, if it were requested in the URL (as in ".json"). This has been fixed with a complete rewrite of the URI Parser.


## Enhancement: Configuration via "variables.framework", ala FW/1

\([ER90](https://github.com/atuttle/Taffy/issues/90)\) Taffy has been refactored to use a single configuration structure, similar to the way that [Framework/1](https://github.com/seancorfield/fw1/) is configured. Just like FW/1, the variable is named `variables.framework`.

```javascript
variables.framework = {
    reloadKey = "reload",
    reloadPassword = "true"
};
```

See also: [[List of all variables.framework settings]].


## Enhancement: Custom Token Regular Expressions

\([ER60](https://github.com/atuttle/Taffy/issues/60)\) You may now specify your own regular expression for tokens instead of using the default regex that Taffy builds for simple named tokens, by following the token name with a colon and then your custom regular expression. Note that, for any token that ends a URI (e.g. `/my/resource/{userid:\d{9}}`), Taffy automatically handles the optional return format specification, even when you use a custom RegEx. For examples of using custom regular expressions for your tokens, see [[Custom Token Regular Expressions]].


## Enhancement: Added the ability to pass data from onTaffyRequest to resources

\([ER57](https://github.com/atuttle/Taffy/issues/57)\) In onTaffyRequest, you can now add keys to the `requestArguments` argument, and they will be passed on to the resource, by name, just like URI tokens and query string parameters.


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
