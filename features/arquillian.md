## Arquillian Test Support

The WildFly Camel test suite uses the WildFly [Arquillian](http://arquillian.org/) managed container. This can connect to an already running WildFly instance or alternatively start up a standalone server instance when needed.

A number of test enrichers have been implemented that allow you to have these WildFly Camel specific types injected into your Arquillian test cases.

```java
@ArquillianResource
CamelContextFactory contextFactory;

@ArquillianResource
CamelContextRegistry contextRegistry;
```
