## camel-file

The [camel-file](http://camel.apache.org/file2.html) component provides access to file systems, allowing files to be processed by any other Camel Components or messages from other components to be saved to disk.

Here is a simple route that prepends the message with 'Hello' and writes the result to a file in WildFly's data directory.

```java
    final String datadir = System.getProperty("jboss.server.data.dir");

    CamelContext camelctx = new DefaultCamelContext();
    camelctx.addRoutes(new RouteBuilder() {
        @Override
        public void configure() throws Exception {
            from("direct:start").transform(body().prepend("Hello ")).
            to("file:" + datadir + "?fileName=camel-file.txt");
        }
    });
```
