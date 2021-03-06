## camel-hl7

The [camel-hl7](http://camel.apache.org/hl7.html) component is used for working with the HL7 MLLP protocol and [HL7 v2](http://www.hl7.org/implement/standards/product_brief.cfm?product_id=185) messages using the [HAPI](http://hl7api.sourceforge.net/) library.

```java
final String msg = "MSH|^~\\&|MYSENDER|MYRECEIVER|MYAPPLICATION||200612211200||QRY^A19|1234|P|2.4\r";
final HL7DataFormat format = new HL7DataFormat();

CamelContext camelctx = new DefaultCamelContext();
camelctx.addRoutes(new RouteBuilder() {
    @Override
    public void configure() throws Exception {
        from("direct:start")
        .marshal(format)
        .unmarshal(format)
        .to("mock:result");
    }
});
camelctx.start();

HapiContext context = new DefaultHapiContext();
Parser p = context.getGenericParser();
Message hapimsg = p.parse(msg);

ProducerTemplate producer = camelctx.createProducerTemplate();
Message result = (Message) producer.requestBody("direct:start", hapimsg);
Assert.assertEquals(hapimsg.toString(), result.toString());
```
