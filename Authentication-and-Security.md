# Authentication and Security

Where authentication and security are concerned, my current recommendation is to use a combination of HTTP Basic Authentication and SSL. You should be able to accomplish this gracefully with `onTaffyRequest`. Currently there is no functionality in Taffy to assist with this, but it is something that we'll consider for future releases.

> Note: HTTP Basic Auth is sent in clear-text, and as such, you should _**NEVER**_ use it in production code without SSL, because it would be trivial for someone sniffing the traffic to see the credentials. (Firesheep)

## OAuth

So far I am only experienced as a user of OAuth, not a developer. For that reason, I haven't gotten around to implementing any OAuth functionality in Taffy either, though it is something I would like to do at some point. If you would like to contribute, please [get in touch with me](http://fusiongrokker.com/page/contact-me) or just fork and send me a pull request. :)