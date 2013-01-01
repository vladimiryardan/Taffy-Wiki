# Authentication and Security

If fine-grain authentication and security are required, my current recommendation is to use a combination of HTTP Basic Authentication and SSL. You should be able to accomplish this gracefully with `onTaffyRequest`. As of Taffy 1.3, the method [`getBasicAuthCredentials()`](https://github.com/atuttle/Taffy/wiki/Index-of-API-Methods) was added to aid in this process.

> Note: HTTP Basic Auth is sent in clear-text, and as such, you should _**NEVER**_ use it in production code without SSL, because it would be trivial for someone sniffing the traffic to see the credentials. ([Firesheep](http://en.wikipedia.org/wiki/Firesheep))

## OAuth

So far I am only experienced as a user of OAuth, not a developer. For that reason, I haven't gotten around to implementing any OAuth functionality in Taffy either, though it is something I would like to do at some point. If you would like to contribute, please [get in touch with me](http://fusiongrokker.com/page/contact-me) or just fork and send me a pull request. :)