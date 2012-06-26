Uploading files sounds a bit tricky, and it can be if you don't know what you're doing, but really it's not that hard. Let's start by looking at what's necessary to upload a file from a regular form post.

## Uploading from a regular form post

Right off the bat, there's something different about the form tag:

```html
<form method="POST" enctype="multipart/form-data" action="upload.cfm">
```

When uploading files, we have to set the `enctype` attribute of the form to "multipart/form-data". This has several affects on the resulting POST request:

* The `Content-Length` header is included
* The `Content-Type` header is set to `multipart/form-data; boundary=[your boundary here]`
  * The boundary string you include in place of `[your boundary here]` is arbitrary and can/should change for every request. A sample from a Webkit post is: "----WebKitFormBoundaryhodA1te7gSzoTcJq".
  * Note that the boundary text in the example starts with 4 dashes. In the body of the request the boundaries (that I saw in my testing) started with 6 dashes. I believe this to indicate that when you apply boundaries, they should begin with 2 dashes, and be followed with the entire contents of your custom boundary. (If your custom boundary starts with 2 dashes, your request body boundaries would start with 4.)
* Instead of a normal query-string like encoding of the form fields, each field is separated from the others by a boundary. So, if I have a text input named "foo" and I enter the string "bar" and submit the form, then one section of the request body will be:

```
------WebKitFormBoundaryhodA1te7gSzoTcJq
Content-Disposition: form-data; name="foo"

bar
------WebKitFormBoundaryhodA1te7gSzoTcJq
```

Note that in addition to the Content-Disposition header for each field that marks that field as form-data and provides its name, a blank line goes between the header and the value of that field. Also note that I included the closing boundary only to illustrate how the request should be composed. Doubling up on boundaries is not required. (However, your request should end with the boundary, and I don't know if it's required or not but none of my tests had a linebreak after the closing boundary.)

If my post had 3 text fields and a file upload, the request might look like this:

```
------WebKitFormBoundaryhodA1te7gSzoTcJq
Content-Disposition: form-data; name="foo1"

bar
------WebKitFormBoundaryhodA1te7gSzoTcJq
Content-Disposition: form-data; name="foo2"

fubar
------WebKitFormBoundaryhodA1te7gSzoTcJq
Content-Disposition: form-data; name="foo3"

how do you know what kind of day it is?
------WebKitFormBoundaryhodA1te7gSzoTcJq
Content-Disposition: form-data; name="img"; filename="myphoto.jpg"
Content-Type: image/jpeg

Ë‡Ã¿Ë‡â€¡  JFIF     H H  Ë‡ÃŒ XPhotoshop 3.0 8BIM          Z   %G            Photo Booth 8BIM %      qydB
8Â»_28Ã·Ã®Å’â‰ˆ!Â Ë‡â€š 4ICC_PROFILE      $appl    mntrRGB XYZ  Ã¿acspAPPL                          Ë†Ã·      â€ -applWÃ‚oÃ²kâ€žÃ»p> 7Î©\:}Ã¹
rXYZ        gXYZ   4    bXYZ   H    wtpt   \    chad   p   ,rTRC   Ãº    gTRC   Â¨    bTRC   Âº    vcgt   Ãƒ    ndin   â€¡   >desc       Ã´cprt   Âº   @mmod   Â¸   (XYZ       [|  4Â«   Â¥XYZ       sâ‰ˆ  â‰¥D   Ä±XYZ       'Ã®      â‰ |XYZ       Ã›P       Ã¦sf32A   â€ºË‡Ë‡Ã›(   Ã­  Ë ÃªË‡Ë‡ËšÂ¢Ë‡Ë‡Ë Â£   â‚¬  Â¿xcurv         3  curv         3  curv         3  vcgt                 V # âˆž j 9 	 Ã  Â¿ Ã² Ã‡ w	e
<snip>
------WebKitFormBoundaryhodA1te7gSzoTcJq
```

Take note of the headers for the image field, they're quite different. And I believe that the content of the image field is simply the file in binary encoding (not converted to Base64 or anything else).

### And your point is?!

All of that is fine and dandy, I hear you saying, but the browser takes care of all of that for me. Why do I care? The answer is that if you're writing, for example, a native phone app and you want to upload photos take on it, then you'll probably have to reimplement some or all of this behavior. If you're not doing anything on the client side but your customers/consumers need help, perhaps send them this link?

## How to accept uploads from your resources

It's actually rather easy! If you'll recall, the field name I gave to my image field in the above hypothetical form post was "img". Now, in a Taffy resource:

```cfm
<cfcomponent extends="taffy.core.resource" taffy:uri="/images">

	<cffunction name="post">
		<cfargument name="img" />
		<cfargument name="foo1" />
		<cfargument name="foo2" />
		<cfargument name="foo3" />
		<cfset var local = StructNew() />

		<cffile
			action="upload"
			destination="#expandPath('/taffy/examples/api_uploadImg/_scratch/')#"
			fileField="img"
			nameConflict="MakeUnique"
			result="local.uploadResult"
			/>

		<cfreturn representationOf( {args: arguments, result: local.uploadResult} ) />

	</cffunction>

</cfcomponent>
```

Yep, simply include the same `<cffile>` upload code you would for a normal form post file upload. _Awesome, right?!_