# Taffy: The Elevator Pitch

Taffy is a free, open-source framework for creating dead-simple REST APIs using ColdFusion CFCs; with a focus on semantically correct URIs, convention over configuration, and harnessing the power of code metadata.

Implementing an API using Taffy requires only a couple of CFCs and can be done in just a few lines of code. See the [[Getting Started]] page for a quick example.

There is also an [[Index of API Methods]] that documents everything that you can do with Taffy, and everything that Taffy does for you.

## Why use Taffy instead of {another front-controller framework}?

**Isn't Taffy just a front-controller ("MVC") framework? Why shouldn't I just use Fusebox / Mach-ii / Model-Glue / etc and some URL rewriting?**

Yes, Taffy is _just another front-controller framework_ for ColdFusion. And yes, it uses a _**similar**_ url-formatting schema to most other frameworks &mdash; a spin on what most of them call "Search Engine Safe" ("SES") URLs. However, Taffy does have some key differences.

First of all, it's designed specifically for building REST web services. There is a subset of core HTTP functionality that is specific to REST that most MVC frameworks don't expose, but Taffy does. There is also no need for a controller because of the simplified process that Taffy takes: a URI + a Verb maps to a specific function, which gets invoked, and its return value is serialized into the requested format before being returned to the client. 

Secondly, this isn't your mom's SES URL formatting. Most MVC frameworks take the simple way out and encode `?name=foo` as `/name/foo`. Instead, with Taffy, you define that `/artist/{artistId}` represents a specific artist, and when the consumer requests `/artist/17`, your **artist** lookup method is passed an argument collection with: `{ artistId: 17 }`. **Specifically:** The string "artistId" is not visible anywhere in the URL.

You _could_ do something similar with another framework and URL rewriting, but defining and maintaining those rewriting rules would be annoying. I wouldn't do it, but you're free to try.

I think that if you give Taffy a try, you'll enjoy its simplicity.