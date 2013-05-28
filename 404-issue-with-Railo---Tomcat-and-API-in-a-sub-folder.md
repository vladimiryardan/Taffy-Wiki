# Known Issue

It's a known issue, and one we have no control over, that **on vanilla Tomcat** (as opposed to the modified version included in Adobe CF10+) **you'll probably find that you get a 404 error if you try to move your api into a sub-folder**. The 404 will complain that the entire URI is not found, such as:

    /api/index.cfm/myResource

... Of course this is not found, because "index.cfm/myResource" isn't a file.

All hope is not lost, however!

## The Fix

In your web.xml, you need to add an additional servlet mapping:

```xml
<servlet-mapping>
  <servlet-name>CFMLServlet</servlet-name>
  <url-pattern>/api/index.cfm/*</url-pattern>
</servlet-mapping>
```

This is because Tomcat doesn't support the use of two wildcards in its mappings. You'll notice that installing ACF or Railo in Tomcat you'll get a web.xml with mappings that have a `url-pattern` of `index.cfm/*`, but unfortunately because of this limitation, you can't change that to `*/index.cfm/*`.

In the xml above you can see that I only have 1 wildcard, but to compensate I've specified the entire path to index.cfm, so that only 1 is needed. If you need to have multiple API's, you'll need a mapping for each index.cfm, and specify the full path of each. (Note that I used `/api/index.cfm` because it matched my example of a 404 for `/api/index.cfm/myResource`... yours should match the location of your index.cfm)