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