When using Taffy's internal, convention-based object factory, all of your resources must go into the `/resources` subfolder of your API. One common request was to allow for further organization within the resources folder, into subfolders. As of Taffy 1.2, this is now possible.

It works seamlessly, meaning "just do it", but it does have one caveat that you should be aware of in case you're also using its auto-wiring functionality to resolve dependencies.

Previously, the "bean name" was the CFC name, minus ".cfc;" but now that you can use multiple subfolders, you could potentially have two resources with the exact same file name in different folders. To prevent a bean name collision, the subfolder names are prepended to the bean name. For example, if you have the files:

 * `/resources/Foo/Widget.cfc`
 * `/resources/Bar/Widget.cfc` and
 * `/resources/Widget.cfc`

Their bean names, respectively, would be:

 * FooWidget
 * BarWidget and
 * Widget

This matters for dependency resolution because it looks for setters containing bean names. For example, if `FooWidget.cfc` contains a method named `setBarWidget`, then the bean named `BarWidget` will be passed to it. Even though the file names are all `Widget.cfc`, they each get unique bean names based on their path.

For this reason, when using subfolders in this way, we recommend Capitalizing the first letter of each subfolder and CFC to improve readability.