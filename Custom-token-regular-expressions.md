When creating your API it is sometimes useful to have similar but distinct URI's or endpoints. For example, you may want to expose course data at a school; and allow listing all courses for a term as: `/courses/2012A`, but also by department, as: `/courses/engineering`.

Previously, if you wanted to put these into two different CFC's you'd run into trouble because their **taffy:uri** patterns would be too similar: `/courses/{term}` and `/courses/{dept}` causes Taffy to throw an exception because it can't tell the two tokens apart.

As of Taffy 1.2, you can now specify a custom regular expression for each token, so that you can guarantee uniqueness.

So, in our example here, we can use this for courses by term: `/courses/{term:\d{4}[A-D]}` and this for courses by department: `/courses/{dept:[A-Za-z0-9]+}`.

**Note:** You can't supply the same regular expression to both CFC's; that's still illegal. `/courses/{dept:\d{4}}` and `/courses/{term:\d{4}}` are still indistinguishable from one another. The Regexes must be unique.