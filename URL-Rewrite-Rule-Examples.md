Rewriting is what enables your API URLs to look like this:
```
http://example.com/api/artists
```
Instead of like this:
```
http://example.com/api/index.cfm/artists
```

This page gives example URL Rewriting rules for the most common Rewriting engines.

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

## IIS URL Rewrite

See the [official documentation](http://www.iis.net/download/urlrewrite) for more information.  This IIS extension works with IIS versions 7.0, 7.5, 8.0 and 8.5.

After installing the IIS extension, be sure to close and reopen the IIS Manager, so the extension can load.  Select the website you wish to configure and finally open the URL Rewrite extension.

Alternately, you can adjust the web.config file located in the root of the website you wish to effect.

### Example 1 - Folder

This example demonstrates how to handle rewriting for an API located in a folder of the website.  Below is a sample URL that would be used to access your API.

`https://www.domain.com/api/artists`

This would be translated to:

`https://www.domain.com/api/index.cfm?endpoint=/artists`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
    <system.webServer>
        <rewrite>
            <rules>
                <rule name="api" stopProcessing="true">
                    <match url="^api/(.*)$" ignoreCase="false" />
                    <action type="Rewrite" url="api/index.cfm?endpoint=/{R:0}" appendQueryString="true" />
                </rule>
            </rules>
        </rewrite>
        <httpErrors existingResponse="PassThrough" />
    </system.webServer>
</configuration>
```

### Example 2 - Subdomain

This example demonstrates how to handle rewriting for an API located at the root of the website.  Below is a sample URL that would be used to access your API.

`https://api.domain.com/artists`

This would be translated to:

`https://api.domain.com/index.cfm?endpoint=/artists`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
    <system.webServer>
        <rewrite>
            <rules>
                <rule name="api" stopProcessing="true">
                    <match url="^(?!index\.cfm)(.*)$" ignoreCase="false" />
                    <action type="Rewrite" url="/index.cfm?endpoint=/{R:0}" appendQueryString="true" />
                </rule>
            </rules>
        </rewrite>
        <httpErrors existingResponse="PassThrough" />
    </system.webServer>
</configuration>
```
