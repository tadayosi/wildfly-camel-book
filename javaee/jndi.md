## Integration with JNDI

JNDI integration is provided by a WildFly specific CamelContext, which is can be obtained like this

```java
InitialContext inictx = new InitialContext();
CamelContextFactory factory = inictx.lookup("java:jboss/camel/CamelContextFactory");
WildFlyCamelContext camelctx = factory.createCamelContext();

```

From a `WildFlyCamelContext` you can obtain a preconfigured Naming Context

```java
Context context = camelctx.getNamingContext();
context.bind("helloBean", new HelloBean());
```

which can then be referenced from Camel routes.

```java
camelctx.addRoutes(new RouteBuilder() {
    @Override
    public void configure() throws Exception {
        from("direct:start").beanRef("helloBean");
    }
});
camelctx.start();
```
