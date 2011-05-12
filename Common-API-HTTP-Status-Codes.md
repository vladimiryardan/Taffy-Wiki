# Common API HTTP Status Codes

HTTP Status Codes are a fundamental concept of REST and the web in general. You should familiarize yourself with [all of them](http://en.wikipedia.org/wiki/List_of_HTTP_Status_Codes), but those listed here are every-day codes that you should probably commit to memory.

## 2xx:Success

* 200 OK -- Everything is fine, I'm returning what you asked for.
* 201 Created -- I've created the resource you submitted

## 3xx: Redirects

* 301 Moved Permanently -- Anything maintaining links to this resource should update the links to the new location
* 302 Found -- Temporarily available at the new location

## 4xx: Client Error

* 400 Error -- General error condition, such as malformed input data
* 401 Unauthorized -- You need to identify yourself before the request will be able to continue
* 403 Forbidden -- You have been identified but do not have permission to access this resource or run the requested action
* 404 Not Found -- The requested resource does not exist

## 5xx: Server Error

* 500 Error -- General or Unkown error
* 503 Service Unavailable -- Usually indicates app server or database is unavailable