_This example is implemented in the folder `examples/api/` (included in the download)._

### Application.cfc and index.cfm

Every API starts with two simple files: `Application.cfc` and `index.cfm`. The latter, `index.cfm` should always be empty. It doesn't _have_ to be empty, but _its contents will be ignored_ so sometimes I'll put in a note for developers that may come behind me so they understand how it works. That just leaves `Application.cfc`.

All that you need in `Application.cfc` for Taffy to work is the following:

```cfm
<cfcomponent extends="taffy.core.api">
	<cfset this.name = "your_api_app_name" />
</cfcomponent>
```

The way that Taffy works is to take over the traditional request event lifecycle. That is, it operates by doing things in the `onApplicationStart()`, `onRequestStart()`, and `onRequest()` methods. This is possible because of the code: `extends="taffy.core.api"`.

If you would like to run some of your own code for these events, Taffy will call a couple of methods that you can optionally define:

* Instead of `onApplicationStart()`, you should define `applicationStartEvent()`.
* Instead of `onRequestStart()`, you should define `requestStartEvent()`.
* You probably shouldn't use `onRequest()`. If you think you need to, you should probably [ask about it on the mailing list](https://groups.google.com/forum/#!forum/taffy-users), and we'll help you figure out if your use-case does need it and how to proceed.

For example, if you need to include a username and password in your `<cfquery>` tags because they aren't stored in your datasource, you can set application variables like this:

```cfm
<cfcomponent extends="taffy.core.api">
	<cfset this.name = "your_api_app_name" />

	<cffunction name="applicationStartEvent">
		<cfset application.dbUser = "username" />
		<cfset application.dbPass = "password" />
	</cffunction>
</cfcomponent>
```

### Next, you need to add a resource.

There are two types of resources: Collections and Members. You can think of Collections like ColdFusion Query objects, and Members like ColdFusion structs. A collection resource represents a collection of member-resource data. The `/students` URI represents the collection of all students, while `/students/12` URI represents a single member of the students collection. It is because of this URI distinction that we separate collections and members into different ColdFusion Components (CFCs).

You can name the CFCs anything you like, but I tend to name mine `thingCollection` and `thingMember` to be consistent and clear. Put them in the `/resources` subfolder of your API. Interested in using sub-folders inside `/resources`? See [[Organizing your resources into subfolders]].

**resources/studentCollection.cfc**

```cfm
<cfcomponent extends="taffy.core.resource" taffy:uri="/students">

	<cffunction name="get">
		<cfset var local = {} />
		<cfquery name="local.qGetStudents">
			select * from students
		</cfquery>
		<cfreturn representationOf( local.qGetStudents ) />
	</cffunction>

</cfcomponent>
```

**resources/studentMember.cfc**

```cfm
<cfcomponent extends="taffy.core.resource" taffy:uri="/students/{studentId}">

	<cffunction name="get">
		<cfargument name="studentId" />
		<cfset var local = {} />
		<cfquery name="local.qGetStudent">
			select * from students
			where studentId = <cfqueryparam cfsqltype="cf_sql_integer" value="#arguments.studentId#" />
		</cfquery>
		<cfreturn representationOf( local.qGetStudent ) />
	</cffunction>

</cfcomponent>
```

The differences between these two CFCs might be sort of subtle:

**The `taffy:uri` element is different.** The collection resource uses the value `/students` and the member resource uses the value `/students/{studentId}`. These determine which CFC will be used to respond to a given URL. The part in curly braces ("{}") is called a token, and you can kind of think of it as a variable. When the CFC method is called, the value that shows up in the URL in the same position as the token (so 12 in the case of `/students/12`) is passed to the function argument that has the same name as the token ("studentId"). This way, `/students/12` and `/students/761` return two distinct records. If you want it to, a URI can contain as many tokens as you like. This is perfectly valid: `/courses/{dept}/{courseNum}/{sectionNum}/students/{studentId}`.

Aside from the query that is run and the URI differences just described, these components are basically identical.

* Because the method is named "get", it will be called when the request verb is GET. If for some reason you don't want to name your method "get", you can name it anything you like and use the function attribute `taffy:verb="get"` to specify that this function should be called for GET requests. The same conventions apply for all other verbs as well.
* Because no other methods are defined in these components requests using verbs other than GET (like POST, PUT, and DELETE) will automatically be refused with an HTTP status of [405 Method Not Allowed](https://github.com/atuttle/Taffy/wiki/Common-API-HTTP-Status-Codes).

**Let's allow student records to be deleted**

To keep the example simple, we're going to ignore security. Depending on your data, chances are good you don't want just anyone deleting it. We'll cover security later.

While you could implement delete for _**an entire collection**_ in a similar manner, it will be a rare, rare API that gives you this sort of freedom. Instead, let's write a custom response to add a little personality to the result:

**in resources/studentCollection.cfc**

```cfm
<cffunction name="delete">
	<cfreturn
		noData()
		.withStatus(405, "Method Not Allowed")
		.withHeaders(
			{"X-FAIL-MESSAGE"="This attempt to delete all students has been added to your permanent record."}
		)
	/>
</cffunction>
```

As previously mentioned, Taffy will automatically deny a DELETE request if you haven't implemented a delete method. But this example does give us an opportunity to introduce a few more bits of the framework without much overhead. Four things are new here: `noData()`, `withStatus()`, `withHeaders()`, and method chaining.

* `noData()` tells Taffy that you want to return an empty response body to the client.
* `withStatus()` sets the HTTP Status Code of the response. In this example we've used 405 to indicate that this method (delete) is not allowed. The second argument is the status text that would accompany the status code. (For example, in "200 OK", "200" is the status code, and "OK" is the status text.)
* `withHeaders()` allows you to return custom headers for the current request. You pass it a structure whose key names will be the header names (usually custom headers should start with an X), and whose values are the header values. Note that the implicit struct notation used above is not compatible with ColdFusion 8; there you must declare and assign values to the structure before passing it to the method.
* Inspired very much by jQuery, Taffy encourages a lot of method chaining. This means simply that you can fit a lot of information, expressively, into one line. This is method chaining: `noData().withStatus().withHeaders()` -- each method in the chain modifys the object returned by the first method in the chain, but then returns that same object, so that the next method in the chain can modify it, and so on, until you stop chaining.

To delete a single student record, you'll want to add a delete method to the student member resource:

**resources/studentMember.cfc**

```cfm
<cffunction name="delete">
	<cfargument name="studentId" />

	<cfset var local = {} />
	<!--- check that the student exists first --->
	<cfquery name="local.qCheckStudentExists">
		select count(studentId) from students
		where studentId = <cfqueryparam cfsqltype="cf_sql_integer" value="#arguments.studentId#" />
	</cfquery>
	<cfif local.qCheckStudentExists.count eq 0>
		<cfreturn noData().withStatus(404, "Not Found") />
	</cfif>

	<!--- student found, delete them --->
	<cfquery name="local.qDeleteStudent">
		delete from students
		where studentId = <cfqueryparam cfsqltype="cf_sql_integer" value="#arguments.studentId#" />
	</cfquery>

	<cfreturn noData() />
</cffunction>
```

In this example you can see that we've added a little bit of input validation. Before performing the delete, we check to see if the record exists, and if not, return 404 Not Found.

### Inserting and Updating works the same way.

You'll write a method that responds for the "POST" (insert) and "PUT" (update) methods, do appropriate input validation, and then run some insert or update sql. Status code 201 is used to indicate a successful insert. The only significant difference is that in some--perhaps most--cases, POST and PUT methods should return the inserted/updated record for confirmation. At a minimum, your post method should set a header to return the created record's ID value (if you're using an identity key) -- I like "X-INSERTED-ID".
