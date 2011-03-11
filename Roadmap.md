## Version 1.1
**Target Release Date:** May 13, 2011 (To be announced at [my cfObjective presentation](http://lanyrd.com/2011/cfobjective/scfzb/))
### Completed
  * New features:
    * Extensive test suite added - helpful for contributors to know they haven't broken anything
    * Added support for HTTP "HEAD" verb
    * Added ability to add custom headers to a response
    * Moved mime-type declaration ("this api supports json,xml,etc") into representation class metadata, instead of api-level configuration.
    * Added a set of Eclipse/CFBuilder snippets to help speed Taffy development (it wasn't fast enough already?!)
    * Added ability to manage representation class with internal/external bean factory, as well as resolving dependencies using the bean factory.
    * Added three more example implementations to the examples folder
    * [\#21](https://github.com/atuttle/taffy/issues/21) - Added global custom headers 
    * [\#20](https://github.com/atuttle/taffy/issues/20) - Added setting for cross-domain resource sharing
    * [\#28](https://github.com/atuttle/taffy/issues/28) - Bean factory throws an exception when requesting a non-existant bean - [Brian Panulla](https://github.com/bpanulla)
    * [\#29](https://github.com/atuttle/taffy/issues/29) - HTTP Method Tunneling  - [Brian Panulla](https://github.com/bpanulla)
  * Bugs fixed:
    * [\#19](https://github.com/atuttle/taffy/issues/19) - JSON input data not properly detected & deserialized into arguments
    * [\#23](https://github.com/atuttle/taffy/issues/24) - Various issues with ColdSpring integration - [Brian Panulla](https://github.com/bpanulla)
### Planned
  * Update documentation
  * Support for binary results, such as image streaming, ala [PlaceKitten](http://www.placekitten.com).
  * [\#18](https://github.com/atuttle/taffy/issues/18) - Overriding the global representation class at the resource/method level can cause errors
  * Tested and supported on Railo and OpenBD
  * Improvements to dashboard, testing, and auto-generated documentation

## Version 1.0
**Released:** [August 23, 2010](http://fusiongrokker.com/post/taffy-a-restful-framework-for-coldfusion)