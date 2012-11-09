In theory, you can use whatever Dependency Injection ("DI") framework you want with Taffy.

> "If you choose not to decide, you still have made a choice." -Rush

If you don't use a 3rd party DI framework, Taffy will use its own, ridiculously small and lightweight DI framework, simply called its "Factory".

Both the Taffy Factory and ColdSpring adhere to the same (imaginary) interface:

* `bool containsBean( string beanId )`
* `any getBean( string beanId )`
* some method to get a list of all bean id's that the DI framework is aware of.
  * ColdSpring uses `string getBeanDefinitionList()`, and Taffy Factory uses `string getBeanList()`.
  * The name/implementation doesn't really matter as we can code around this. The important thing is that we can get a list of all beans. Even better if we can filter that list by a parent class. E.g. get beans that (at some point in their inheritance chain) extend from `taffy.core.resource`. If filtering isn't provided, we get all beans and filter manually.

That's it! If your DI framework implements that interface, it's Taffy-compatibile.