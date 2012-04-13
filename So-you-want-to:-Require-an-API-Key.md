_This example is implemented in the folder `examples/api_requireApiKey/` (included in the download)._

Taffy exposes a few points in the request lifecycle of Application.cfc to you via optional methods that you can implement. One of them is `onTaffyRequest`, which is called after the request has been parsed to figure out what it is that's being requested. What your `onTaffyRequest` method returns decides how Taffy will continue.

This is the method signature that Taffy calls, in your Application.cfc: `onTaffyRequest(verb, cfc, requestArguments, mimeExt, headers);`

* **verb:** (string) the HTTP verb that the consumer used. GET, POST, PUT, DELETE, etc.
* **cfc:** (string) the cfc that would be used to service the request.
* **requestArguments:** (struct) the arguments that will be sent to the resource method.
* **mimeExt:** (string) the requested "extension" of the return format, so if they want json, it's `json` (as opposed to `application/json`).
* **headers:** (struct) the request headers.

If you implement `onTaffyRequest`, you can return one of two ways:

* **Return TRUE** to allow the request to continue as the consumer intended.
* **Return a Representation Object** to abort the request and return the contents of your representation object to the consumer.

If you choose not to implement `onTaffyRequest`, the default implementation always returns true.

You can use this to require an API key. Here's one such approach to do that. Put this in your Application.cfc:

```cfm
function onTaffyRequest(verb, cfc, requestArguments, mimeExt){

	if(not structKeyExists(arguments.requestArguments, "apiKey")){
		//unauthorized because they haven't included their API key
		return newRepresentation().noData().withStatus(401, "API Key Required");
	}

	//api key found
	return true;
}
```

This code checks for a request argument named "apiKey" (i.e. a `GET` request for `/foo?apiKey=abc123`), and if it's not found, returns a blank response body with status code 401. The `newRepresentation()` method creates a new instance of your default representation class, then you can either use `noData()` or if you want to have a response body, use `setData()` to pass in the data to send back to the consumer. Aside from the `newRepresentation()` and `setData()` methods, this should feel very similar to returning data in your resources. That's essentially what you're doing; just short-circuiting the request.