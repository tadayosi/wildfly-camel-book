## Integration with JAX-WS

WebService support is provided through the [camel-cxf](http://camel.apache.org/cxf.html) component which integrates with the WildFly WebServices subsystem that also uses [Apache CXF](http://cxf.apache.org/).


### JAX-WS CXF Producer
The following code example uses CXF to consume a web service which has been deployed by the [WildFly web services subsystem](https://docs.jboss.org/author/display/WFLY8/JAX-WS+User+Guide).

#### JAX-WS web service
The following simple web service has a simple 'greet' method which will concatenate two string arguments together
and return them.

When the WildFly web service subsystem detects classes containing JAX-WS annotations, it bootstraps a CXF endpoint. In this example
the service endpoint will be located at http://hostname:port/context-root/greeting.

```java
// Service interface
@WebService(name = "greeting")
public interface GreetingService {
    @WebMethod(operationName = "greet", action = "urn:greet")
    String greet(@WebParam(name = "message") String message, @WebParam(name = "name") String name);
}

// Service implementation
public class GreetingServiceImpl implements GreetingService{
    public String greet(String message, String name) {
        return message + " " + name ;
    }
}
```

#### Camel route configuration
This RouteBuilder configures a CXF producer endpoint which will consume the 'greeting' web service defined above. CDI in conjunction with the camel-cdi component
is used to bootstrap the RouteBuilder and CamelContext.
```java
@Startup
@ApplicationScoped
@ContextName("cxf-camel-context")
public class CxfRouteBuilder extends RouteBuilder {

    @Override
    public void configure() throws Exception {
        from("direct:start")
        .to("cxf://http://localhost:8080/example-camel-cxf/greeting?serviceClass=" + GreetingService.class.getName());
    }
}
```

The greeting web service 'greet' requires two parameters. These can be supplied to the above route by way of a `ProducerTemplate`.
The web service method argument values are configured by constructing an object array which is passed as the exchange body.

```java
String message = "Hello"
String name = "Kermit"

ProducerTemplate producer = camelContext.createProducerTemplate();
Object[] serviceParams = new Object[] {message, name};
String result = producer.requestBody("direct:start", serviceParams, String.class);
```

### Camel CXF JAX-WS Consumer
```xml
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:cxf="http://camel.apache.org/schema/cxf"
       xsi:schemaLocation="
        http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
        http://camel.apache.org/schema/cxf http://camel.apache.org/schema/cxf/camel-cxf.xsd
        http://camel.apache.org/schema/spring http://camel.apache.org/schema/spring/camel-spring.xsd">

    <cxf:cxfEndpoint id="cxfConsumer"
                     address="http://localhost:8080/webservices/greeting"
                     serviceClass="org.wildfly.camel.examples.cxf.jaxws.GreetingService" />

    <camelContext id="cxfws-camel-context" xmlns="http://camel.apache.org/schema/spring">
        <route>
            <from uri="cxf:bean:cxfConsumer" />
            <to uri="log:ws" />
        </route>
    </camelContext>

</beans>
```

### JAX-WS Consumer with CamelProxy
JAX-WS consumer endpoints can be configured using the [CamelProxy](http://camel.apache.org/using-camelproxy.html). The proxy enables
you to proxy a producer sending to an Endpoint by a regular interface. When clients invoke methods on this interface, the proxy performs a request / reply to a specified endpoint.

Below is an example JAX-WS web service interface and implementation.

```java
// Service interface
@WebService(name = "greeting")
public interface GreetingService {
    @WebMethod(operationName = "greet", action = "urn:greet")
    String greet(@WebParam(name = "name") String name);

    @WebMethod(operationName = "greetWithMessage", action = "urn:greetWithMessage")
    String greetWithMessage(@WebParam(name = "message") String message, @WebParam(name = "name") String name);
}

// Service implementation
@WebService(serviceName="greeting", endpointInterface = "org.wildfly.camel.examples.jaxws.GreetingService")
public class GreetingServiceImpl {
    @Produce(uri="direct:start")
    GreetingService greetingService;

    @WebMethod(operationName = "greet")
    public String greet(@WebParam(name = "name") String name) {
        return greetingService.greet(name);
    }

    @WebMethod(operationName = "greetWithMessage")
    public String greetWithMessage(@WebParam(name = "message") String message, @WebParam(name = "name") String name) {
        return greetingService.greetWithMessage(message, name);
    }
}
```
Notice in the above code example that `GreetingServiceImpl` delegates all method calls to a greetingService object which has been annotated
with `@Produce`. This annotation is important as it configures a proxy for the `direct:start` endpoint against the `GreetingService` interface. Whenever any of the web service methods are invoked by clients, the `direct:start` camel route is triggered.

The RouteBuilder class implements logic for each web service method invocation.

```java
@Startup
@ApplicationScoped
@ContextName("jaxws-camel-context")
public class JaxwsRouteBuilder extends RouteBuilder {
    @Override
    public void configure() throws Exception {
        from("direct:start")
        .process(new Processor() {
            @Override
            public void process(Exchange exchange) throws Exception {
                BeanInvocation beanInvocation = exchange.getIn().getBody(BeanInvocation.class);
                String methodName = beanInvocation.getMethod().getName();

                if(methodName.equals("greet")) {
                    String name = exchange.getIn().getBody(String.class);
                    exchange.getOut().setBody("Hello " + name);
                } else if(methodName.equals("greetWithMessage")) {
                    String message = (String) beanInvocation.getArgs()[0];
                    String name = (String) beanInvocation.getArgs()[1];
                    exchange.getOut().setBody(message + " " + name);
                } else {
                    throw new IllegalStateException("Unknown method invocation " + methodName);
                }
            }
        });
    }
```

In the above RouteBuilder a `Processor` handles web service method invocations that have been proxied through the `direct:start` endpoint.
The exchange message body will be an instance of `BeanInvocation`. This can be used to determine which web service method was invoked and
what arguments were passed to it. In this example some simple logic is used to return results to the client based on the name of the method that
was called.

### Security

Refer to the [JAX-WS security section](../security/jaxws.md).

### Code examples on GitHub

Example JAX-WS applications are available on GitHub.

* [Camel CXF application](https://github.com/wildfly-extras/wildfly-camel/tree/{{ book.version }}/examples/camel-cxf-jaxws)
