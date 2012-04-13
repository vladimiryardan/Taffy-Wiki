# So you want to... (Index)

This page lists a bunch of tasks you might want to complete while coding with Taffy, and links to subpages explaining how you can accomplish each of them. It was created as part of Taffy 1.1, so directions here may not always work for Taffy 1.0.

## Index:

1. [Create a dead-simple CRUD (Create, Read, Update, Delete) API for one resource (aka table of data)](https://github.com/atuttle/Taffy/wiki/So-you-want-to:-Create-a-dead-simple-CRUD-API)
1. [Serialize your data to a format other than the default of JSON](https://github.com/atuttle/Taffy/wiki/So-you-want-to:-Serialize-data-to-a-different-data-type)
1. [Serialize your return data to multiple formats (json, xml, ...)](https://github.com/atuttle/Taffy/wiki/So-you-want-to:-Support-returning-multiple-formats)
1. Require an API key
1. Rate-limit access to your API
1. Use JSONUtil instead of ColdFusion's native JSON serialization
1. Share application variables between your API and your consumer-facing application
1. Use a bean factory (like ColdSpring) to resolve dependencies of your resources (like configuration without breaking encapsulation)
1. Use a bean factory (like ColdSpring) to completely manage resources
1. Use ColdSpring AOP advice for your resources
1. Write your components using ColdFusion 9+ "script component" syntax


## Require an API key

_This example is fully implemented in the folder `examples/api_requireApiKey/`._

Taffy exposes a few points in the request lifecycle to you via methods. One of them is `onTaffyRequest`, which is called after the request has been parsed to figure out what it is that's being requested. What your `onTaffyRequest` method returns decides how Taffy will continue.

This is the method signature that Taffy calls, in your Application.cfc: `onTaffyRequest(verb, cfc, requestArguments, mimeExt, headers);`

* **verb:** (string) the verb that the consumer used. GET, POST, PUT, DELETE, etc.
* **cfc:** (string) the cfc that would be used to service the request.
* **requestArguments:** (struct) the arguments that will be sent to the resource method.
* **mimeExt:** (string) the requested "extension" of the return format, so if they want json, it's `json` (as opposed to `application/json`).
* **headers:** (struct) the request headers.

If you implement `onTaffyRequest`, you can return one of two ways:

* **Return TRUE** to allow the request to continue as the consumer intended.
* **Return a Representation Object** to abort the request and return whatever your returned representation object.

If you choose not to implement `onTaffyRequest`, the default implementation always returns true.

You can use this to require an API key. Here's one such approach to do that. Put this in your Application.cfc:

```cfm
function onTaffyRequest(verb, cfc, requestArguments, mimeExt){

	if(not structKeyExists(arguments.requestArguments, "apiKey")){
		return newRepresentation().noData().withStatus(401);//unauthorized because they haven't included their API key
	}

	//api key found
	return true;
}
```

This code checks for a request argument named "apiKey" (i.e. a `GET` request for `/foo?apiKey=abc123`), and if it's not found, returns a blank response body with status code 401. The `newRepresentation` method creates a new instance of your representation class, then you can either use `noData()` or if you want to have a response body, use `setData()` to pass in the data to send back to the consumer. Aside from the `newRepresentation()` and `setData()` methods, this should feel very similar to returning data in your resources. That's essentially what you're doing; just short-circuiting the request.

## Rate-limit access to your API



## Use JSONUtil instead of ColdFusion's native JSON serialization



## Share application variables between your API and your consumer-facing application



## Use a bean factory (like ColdSpring) to resolve dependencies of your resources (like configuration without breaking encapsulation)



## Use a bean factory (like ColdSpring) to completely manage resources



## Use ColdSpring AOP advice for your resources



## Write your components using ColdFusion 9+ "script component" syntax