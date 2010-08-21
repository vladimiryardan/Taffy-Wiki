 >*Warning:* Documentation Currently Outdated<br/><br/>As this product is pre- version 1.0, I am still making breaking changes and not going out of my way to preserve backward compatibility as I come up with better ideas for the ways things can work. The documentation is currently a little bit outdated, but will be updated as part of the 1.0 release.<br/><br/>While the documentation is out of date, the example applications will show working code of varying kinds; as these are my test cases, and I don't develop the core without updating the tests.<br/><br/>_Thanks for your patience as I improve Taffy!_

# Intro

Hi there. If you're looking for some high level information on Taffy, you should check out the [project homepage](http://atuttle.github.com/Taffy/)

To see exactly what's required to implement a REST API with Taffy, check out the [[Getting Started]] page.

There is also an [[Index of API Methods]] that documents what each does and how to work with it.

## Why use Taffy instead of {another front-controller framework}?

**Isn't Taffy just a front-controller framework? Why shouldn't I just use Fusebox / Mach-ii / Model-Glue / etc and some URL rewriting?**

Yes, Taffy is _just another front-controller framework_ for ColdFusion. And yes, it uses a _*similar*_ url-formatting schema to most other frameworks -- what they call "SES (Search Engine Safe) URLs." However, Taffy does have some key differences.

First of all, it's designed specifically for building REST web services. It doesn't try to solve any MVC problems, because there is no styling - just input and output. Secondly, this isn't your mom's SES URL formatting. We're not just encoding `?name=foo` as `/name/foo`. Instead, you define that `/artist/{artistId}` represents a specific artist, and when the consumer requests `/artist/17`, your *artist* lookup method is passed an argument collection with: `{ artistId: 17 }`. The "artistId" portion is not visible anywhere in the URL.

You _could_ do something similar with URL rewriting, but defining and maintaining those rewriting rules would be annoying. I wouldn't recommend it, but you're free to try.

I think that if you give Taffy a try, you'll enjoy its simplicity.