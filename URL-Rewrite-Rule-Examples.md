>**Warning:** Documentation Currently Outdated<br/><br/>As this product is pre- version 1.0, I am still making breaking changes and not going out of my way to preserve backward compatibility as I come up with better ideas for the ways things can work. The documentation is currently a little bit outdated, but will be updated as part of the 1.0 release.<br/><br/>While the documentation is out of date, the example applications will show working code of varying kinds; as these are my test cases, and I don't develop the core without updating the tests.<br/><br/>_Thanks for your patience as I improve Taffy!_

This page gives example URL Rewriting rules for the most common Rewriting engines. All examples assume that your API exists at: `http://example.com/api/index.cfm`, and the resulting URL should resemble: `http://example.com/api/artists` (where "/artists" is the resource being requested)

## Apache mod_rewrite

See the [official documentation](http://httpd.apache.org/docs/2.2/mod/mod_rewrite.html) for more information.

```apache
    RewriteRule ^/api/(.*) http://example.com/api/index.cfm/$1
```

## IIRF (Ionic's ISAPI Rewrite Filter)

See the [official documentation](http://cheeso.members.winisp.net/Iirf21Help/frames.htm) for more information.

```apache
    RewriteRule ^/api/(.*)$ /api/index.cfm/$1
```

## ISAPI Rewrite (Helicon)

See the [official documentation](http://www.isapirewrite.com/docs/) for more information.

```apache
    ?
```