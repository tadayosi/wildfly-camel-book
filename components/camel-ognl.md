## camel-ognl

OGNL expression support in Camel is provided by the [camel-ognl](http://camel.apache.org/ognl.html) component.


## Simple Camel OGNL Use Case
```java
CamelContext camelContext = new DefaultCamelContext();

camelContext.addRoutes(new RouteBuilder() {
    @Override
    public void configure() throws Exception {
        from("direct:start")
            .choice()
                .when()
                    .ognl("request.body.name == 'Kermit'").transform(simple("Hello ${body.name}"))
                .otherwise()
                    .to("mock:dlq");
    }
});

camelContext.start();

Person person = new Person();
person.setName("Kermit");

ProducerTemplate producer = camelContext.createProducerTemplate();
String result = producer.requestBody("direct:start", person, String.class);

Assert.assertEquals("Hello Kermit", result);

camelContext.stop();
```
