This page only covers future releases. For the history of previous releases, see the [[Releases]] page.

## Version 1.1
**Target Release Date:** ?? ??, 2011

### Completed

* New features:
  * Extensive test suite added - helpful for contributors to know they haven't broken anything
    * Update test suite to use Tags instead of script so it can be run on any CFML engine (What good are tests if you can't use them to test?)
  * Added support for HTTP "HEAD" verb
  * Added ability to add custom headers to a response
  * Moved mime-type declaration ("this api supports json,xml,etc") into representation class metadata, instead of api-level configuration.
  * Added a set of Eclipse/CFBuilder snippets to help speed Taffy development (it wasn't fast enough already?!)
  * Added ability to manage representation class with internal/external bean factory, as well as resolving dependencies using the bean factory.
  * Added several more example implementations to the examples folder
  * [\#21](https://github.com/atuttle/taffy/issues/21) - Added global custom headers 
  * [\#20](https://github.com/atuttle/taffy/issues/20) - Added setting for cross-domain resource sharing
  * [\#28](https://github.com/atuttle/taffy/issues/28) - Bean factory now throws an exception when requesting a non-existant bean - Thanks to [Brian Panulla](https://github.com/bpanulla)
  * [\#29](https://github.com/atuttle/taffy/issues/29) - HTTP Method Tunneling  - Thanks to [Brian Panulla](https://github.com/bpanulla)
  * Added example of rate limiting using onTaffyRequest
  * Support for direct streaming of binary results, such as image streaming, ala [PlaceKitten](http://www.placekitten.com), or a generated PDF (examples included).
  * Fix issues on startup if Application context is shared with another app & Taffy doesn't get properly initialized.
  * Update test suite to use Tags instead of script so it can be run on any CFML engine (What good are tests if you can't use them to test?)
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
* Updated documentation

### Planned
* Fix issues on startup if Application context is shared with another app & Taffy doesn't get properly initialized.

## Version 1.2
**Target Release Date:** TBD
* Tested and supported on Railo and OpenBD

### Planned
* [\#18](https://github.com/atuttle/taffy/issues/18) - Overriding the global representation class at the resource/method level can cause errors
* [\#48](https://github.com/atuttle/taffy/issues/48) - Allow resources to be organized into subfolders
* Auto-generated public-facing documentation at `?docs`
* Improvements to dashboard, testing, and auto-generated documentation
* Pre-return hook for caching
* Helper methods for http basic-auth
* Extensibility to support custom input formats (in addition to form fields, url-encoded, and json)
