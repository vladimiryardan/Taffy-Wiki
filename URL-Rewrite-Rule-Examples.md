This page gives example URL Rewriting rules for the most common Rewriting engines. All examples assume that your API exists at: `http://example.com/api/index.cfm`, and the resulting URL should resemble: `http://example.com/api/artists` (where "/artists" is the resource being requested)

## Apache mod_rewrite

See the [official documentation](http://httpd.apache.org/docs/2.2/mod/mod_rewrite.html) for more information.

```apache
    RewriteCond %{REQUEST_FILENAME} !-f
    RewriteRule ^/api/(.*) /api/index.cfm/$1
```

If you installed Tomcat _separately_ from ColdFusion, you need to make use of [endpointUrlParam](http://docs.taffy.io/2.2.4/#endpointURLParam) **\*or\*** add Servelet Mappings for each API you create [as described here](http://docs.taffy.io/2.2.4/#404-when-your-API-is-in-a-subdirectory). 

The Tomcat flavor installed by the CF installer is unaffected: If that's where you got Tomcat, you can skip this. 

If you choose to use endpointUrlParam with the default config, your rewrite rules should look like this:

```apache
    RewriteCond %{REQUEST_FILENAME} /api/
    RewriteCond %{REQUEST_FILENAME} !/api/index.cfm
    RewriteCond %{REQUEST_FILENAME} !/api/resources
    RewriteRule ^/api/(.*) /api/index.cfm?endpoint=/$1 [PT]
```
where `/api/` is the path from your web root to where ever you installed Taffy i.e. it should have a `resources/` subfolder. You'll still need the mappings in Application.cfc in that folder to match, of course.

## IIRF (Ionic's ISAPI Rewrite Filter)

See the [official documentation](http://cheeso.members.winisp.net/Iirf21Help/frames.htm) for more information.

```apache
    RewriteRule ^/api/(.*)$ /\api/index.cfm/$1
```

>*Note:* As far as I can tell, the backslash in the 2nd half of the rule ("\api") SHOULD NOT be necessary, but I've seen more than one report that someone's rule didn't work without it, but did after adding it. _I don't understand why it's necessary_, but I'm documenting it here for your benefit!

## ISAPI Rewrite (Helicon)

See the [official documentation](http://www.isapirewrite.com/docs/) for more information.

```apache
    ?
```

> **Be awesome:** [Contribute to the wiki](http://fusiongrokker.com/post/how-you-can-contribute-to-taffy-documentation) and document what this syntax should be!

## IIS7 web.config

See the [official documentation](http://www.iis.net/download/urlrewrite) for more information.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
    <system.webServer>
        <rewrite>
            <rules>
                <rule>
                    <match url="^api/([.*])$" />
                    <action type="Rewrite" url="index.cfm/{R:1}" />
                </rule>
            </rules>
        </rewrite>
    </system.webServer>
</configuration>
```