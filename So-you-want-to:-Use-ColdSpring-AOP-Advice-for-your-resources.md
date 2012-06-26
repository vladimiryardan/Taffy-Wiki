There's no real secret to this. It sounds complicated, but it isn't! (_Assuming you're already familiar with AOP in CS_)

You'll want to start by getting ColdSpring running for your API. Don't even bother with AOP until your resources are [completely managed from within ColdSpring][1].

Then you just... apply some AOP advice. _It's really that simple._ When ColdSpring returns the resource that Taffy requests from it, your resource will be wrapped in an AOP proxy, so all of the AOP magic will be built in. I'm not going to go into any description of what AOP accomplishes or when or why to use it, or even how it works. [That's all been covered very well by now][2], so from here on out we're assuming you have a basic understanding of AOP!

But just for fun, let's implement some!

## AOP Advice

It's a bit of a tired example, but let's say you wanted to log all API access. We'll start by creating our logging advice (adapted from [the AOP documentation][2]):

```cfm
<cfcomponent extends="coldspring.aop.MethodInterceptor">

	<cffunction name="init" returntype="any" output="false" access="public" hint="Constructor">
		<cfreturn this />
	</cffunction>

	<cffunction name="invokeMethod">
		<cfargument name="methodInvocation" type="coldspring.aop.MethodInvocation" required="true" />
		<cfset var local =  StructNew() />

		<cfset local.logData = StructNew() />
		<cfset local.logData.arguments = StructCopy(arguments.methodInvocation.getArguments()) />
		<cfset local.logData.method = arguments.methodInvocation.getMethod().getMethodName() />
		<cfset local.logData.timestamp = now() />
		<cflog application="false" file="API_ACCESS" text="#serializeJson(local.logData)#" />

		<!--- Proceed with the call to the underlying Taffy Resource CFC. --->
		<cfreturn arguments.methodInvocation.proceed() />
	</cffunction>

</cfcomponent>
```

Then we simply have to define it in our coldspring.xml as an available advice and create an advisor with it:

```xml
<!DOCTYPE beans PUBLIC "-//SPRING//DTD BEAN//EN" "http://www.springframework.org/dtd/spring-beans.dtd">
<beans>

	<bean id="loggingAdvice" class="yourapp.com.LoggingAdvice" />

	<bean id="loggingAdvisor" class="coldspring.aop.support.NamedMethodPointcutAdvisor">
		<property name="advice">
			<ref bean="loggingAdvice" />
		</property>
		<property name="mappedNames">
			<value>*</value>
		</property>
	</bean>

</beans>
```

... and apply it to our resources:

```xml
<!DOCTYPE beans PUBLIC "-//SPRING//DTD BEAN//EN" "http://www.springframework.org/dtd/spring-beans.dtd">
<beans>

	<bean id="loggingAdvice" class="yourapp.com.LoggingAdvice" />

	<bean id="loggingAdvisor" class="coldspring.aop.support.NamedMethodPointcutAdvisor">
		<property name="advice">
			<ref bean="loggingAdvice" />
		</property>
		<property name="mappedNames">
			<value>*</value>
		</property>
	</bean>

	<!-- repeat this XML block for each resource -->
	<bean id="someCollection" class="coldspring.aop.framework.ProxyFactoryBean">
		<property name="target">
			<bean class="yourapp.com.SomeCollection" />
		</property>
		<property name="interceptorNames">
			<list>
				<value>loggingAdvisor</value>
			</list>
		</property>
	</bean>

</beans>
```

You'll repeat the first block of XML (the one with class `coldspring.aop.framework.ProxyFactoryBean`) once for each Taffy resource that you want to wrap with the logging service; but you don't have to repeat the Logging Advice or Logging Advisor blocks.

[1]: https://github.com/atuttle/Taffy/wiki/So-you-want-to:-use-an-external-bean-factory-like-coldspring-to-completely-manage-resources
[2]: http://coldspringframework.org/coldspring/examples/quickstart/index.cfm?page=aop
