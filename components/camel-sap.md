## camel-sap

[camel-sap](https://access.redhat.com/documentation/en-US/Red_Hat_JBoss_Fuse/6.2/html/Apache_Camel_Component_Reference/SAP.html) is a package consisting of ten different SAP components.

There are remote function call (RFC) components that support the sRFC, tRFC, and qRFC protocols; and there are IDoc components that facilitate communication using messages in IDoc format. The component uses the SAP Java Connector (SAP JCo) library to facilitate bidirectional communication with SAP and the SAP IDoc library to facilitate the transmission of documents in the Intermediate Document (IDoc) format.

> **IMPORTANT**: A prerequisite for using the Camel SAP component is that the SAP Java Connector (SAP JCo) libraries and the SAP IDoc library are available for the WildFly Camel Subsytem to use. **The subsystem distribution does not include these libraries**. You will need access to the [SAP Service Marketplace](http://service.sap.com/connectors) in order to obtain them.

### Configuring the camel-sap module

Before using camel-sap, an additional WildFly / EAP modules needs to be configured so that the subsystem can discover the new functionality.

Your application server should be stopped before making the following changes.

#### Modify module com.sap.conn.jco

Download the SAP JCo libraries and the SAP IDoc library from the [SAP Service Marketplace](http://service.sap.com/connectors)), making sure to choose the appropriate library versions for your operating system.

1. From a terminal session, change into the application server installation root directory
2. Change into directory `modules/system/layers/fuse`
3. Copy `sapjco3.jar` and `sapidoc3.jar` into `com/sap/conn/jco/main`
4. Create an appropriate native library directory within `com/sap/conn/jco/main`. For example, if your target platform is Linux x86_64 you would create `lib/linux-x86_64`. For information relating to other operating systems and architectures see the [WildFly Native Libraries documentation](https://docs.jboss.org/author/display/MODULES/Native+Libraries)
5. Copy the JCO native library (i.e the `.so`, `.dll` or `.jnilib file`) into the native library directory you created in the previous step

The file and directory structure for module `com.sap.conn.jco` should now look something like this:

```
com/sap/conn/jco
com/sap/conn/jco/main
com/sap/conn/jco/main/sapidoc3.jar
com/sap/conn/jco/main/sapjco3.jar
com/sap/conn/jco/main/module.xml
com/sap/conn/jco/main/lib
com/sap/conn/jco/main/lib/linux-x86_64
com/sap/conn/jco/main/lib/linux-x86_64/libsapjco3.so
```

#### Activate camel-sap component module

1. From a terminal session change into the application server installation root directory
2. Change into directory `modules/system/layers/fuse/org/wildfly/camel/extras/main`
3. Edit `module.xml` and uncomment the `org.fusesource.camel.component.sap` module dependency. Save the modifed file when finished

The module definition should look like this:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<module xmlns="urn:jboss:module:1.1" name="org.wildfly.camel.extras">
    <dependencies>
        <module name="org.fusesource.camel.component.sap" export="true" services="export" />
    </dependencies>
</module>
```

Start your application server and camel-sap will be available for use.

### Example SAP Camel Route

This example uses XML files containing serialized SAP requests to query Customer records in the Flight Data Application within SAP.

These files are consumed by Camel and their contents are then converted to string message bodies. These messages are then routed to an `sap-srfc-destination` endpoint which converts and sends them to SAP as `BAPI_FLCUST_GETLIST` requests to query Customer records.

First we configure a destination data store.
```java
DestinationData destinationData = new DestinationDataImpl();
destinationData.setAshost("example.com");
destinationData.setSysnr("00");
destinationData.setClient("000");
destinationData.setUser("user");
destinationData.setPasswd("password");
destinationData.setLang("en");

Map<String, DestinationData> destinationDataStore = new HashMap<>();
destinationDataStore.put("quickstartDest", destinationData);
```

Next wire the destinationDataStore into a SapConnectionConfiguration.
```java
SapConnectionConfiguration configuration = new SapConnectionConfiguration();
configuration.setDestinationDataStore(destinationDataStore);
```

The Camel RouteBuilder looks like this.

```java
@ApplicationScoped
@ContextName("camel-sap-cdi-context")
@Startup
public class SAPRouteBuilder extends RouteBuilder {
    @Override
    public void configure() throws Exception {
      from("file:src/data")
      .convertBodyTo(String.class)
      .to("log:sap?showAll=true")
      .to("sap-srfc-destination:quickstartDest:BAPI_FLCUST_GETLIST")
      .to("log:sap?showAll=true");
    }
}
```

The XML request looks like the following and is consumed from a file within src/data.
```xml
<?xml version="1.0" encoding="ASCII"?>
<!-- NOTE: Replace 'XXX' with the SID of your SAP instance -->
<BAPI_FLCUST_GETLIST:Request xmlns:BAPI_FLCUST_GETLIST="http://sap.fusesource.org/rfc/XXX/BAPI_FLCUST_GETLIST" CUSTOMER_NAME="*" MAX_ROWS="10" WEB_USER="*"/>
```

When the Camel route runs, the request XML data is consumed from src/data and sent to SAP as a `BAPI_FLCUST_GETLIST` request.  The results of the SAP request are output to the console.
You should see an XML structure like the following containing customer records.

```xml
<?xml version="1.0" encoding="ASCII"?>
<BAPI_FLCUST_GETLIST:Response xmlns:BAPI_FLCUST_GETLIST="http://sap.fusesource.org/rfc/JBF/BAPI_FLCUST_GETLIST">
<CUSTOMER_LIST>
    <row CUSTOMERID="00004715" CUSTNAME="Fred Flintstone" FORM="Mr." STREET="123 Flintstone Lane" POBOX="" POSTCODE="01234" CITY="Bedrock" COUNTR="US" COUNTR_ISO="US" REGION="" PHONE="800-555-1212" EMAIL=""/>
    <row CUSTOMERID="00004716" CUSTNAME="Wilma Flintstone" FORM="Mr." STREET="123 Flintstone Lane" POBOX="" POSTCODE="01234" CITY="Bedrock" COUNTR="US" COUNTR_ISO="US" REGION="" PHONE="800-555-1212" EMAIL=""/>
</CUSTOMER_LIST>
<CUSTOMER_RANGE/>
<EXTENSION_IN/>
<EXTENSION_OUT/>
<RETURN>
    <row TYPE="S" ID="BC_IBF" NUMBER="000" MESSAGE="Method was executed successfully" LOG_NO="" LOG_MSG_NO="000000" MESSAGE_V1="" MESSAGE_V2="" MESSAGE_V3="" MESSAGE_V4="" PARAMETER="" FIELD="" SYSTEM="DEVQKCLNT"/>
</RETURN>
</BAPI_FLCUST_GETLIST:Response>
```

### Further Reading

The example above only scratches the surface of the functionality provided by the camel-sap component. For comprehensive component documentation visit
the [Camel SAP Component Reference](https://access.redhat.com/documentation/en-US/Red_Hat_JBoss_Fuse/6.2/html/Apache_Camel_Component_Reference/SAP.html).
