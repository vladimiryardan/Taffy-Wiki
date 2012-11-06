## Bug Fix: File Uploads did not work on Railo

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
