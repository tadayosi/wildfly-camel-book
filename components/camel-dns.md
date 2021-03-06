## camel-dns

The [camel-dns](http://camel.apache.org/dns.html) component provides the ability to:

* Resolve a domain by its ip
* Lookup information about a domain
* Run DNS queries

```java
  CamelContext camelctx = new DefaultCamelContext();
  camelctx.addRoutes(new RouteBuilder() {
      @Override
      public void configure() throws Exception {
          from("direct:start")
          .to("dns:lookup");
      }
  });

  ProducerTemplate producer = camelctx.createProducerTemplate();
  Record[] record = producer.requestBodyAndHeader("direct:start", null, DnsConstants.DNS_NAME, "wildfly.org", Record[].class);
```
