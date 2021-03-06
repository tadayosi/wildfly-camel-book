# Getting Started

This chapter takes you through the first steps of getting WildFly-Camel and provides the initial pointers to get up and running.

## Download the Distribution

WildFly Camel is distributed as

1. WildFly Patch - [wildfly-camel-patch](https://github.com/wildflyext/wildfly-camel/releases)
2. Docker Image - [wildflyext/wildfly-camel](https://registry.hub.docker.com/u/wildflyext/wildfly-camel/)


## Installing the Camel Subsystem

Simply unpack the provided patch into the a supported WildFly installation. For possible WildFly target versions, see the [compatibility page](compatibility.md)

## Standalone Server

In your WildFly home directory run ...

```
$ bin/standalone.sh -c standalone-camel.xml
...
10:50:02,833 INFO  [org.jboss.as.server] (ServerService Thread Pool -- 31) JBAS018559: Deployed "wildfly-camel.war" (runtime-name : "wildfly-camel.war")
10:50:02,834 INFO  [org.jboss.as.server] (ServerService Thread Pool -- 31) JBAS018559: Deployed "hawtio.war" (runtime-name : "hawtio.war")
10:50:02,848 INFO  [org.jboss.as] (Controller Boot Thread) JBAS015961: Http management interface listening on http://127.0.0.1:9990/management
10:50:02,849 INFO  [org.jboss.as] (Controller Boot Thread) JBAS015951: Admin console listening on http://127.0.0.1:9990
10:50:02,849 INFO  [org.jboss.as] (Controller Boot Thread) JBAS015874: WildFly 8.1.0.Final "Kenny" started in 10804ms
```

## Domain Mode

Similarly, for the WildFly Domain Mode run ...  

```
$ bin/domain.sh -c domain-camel.xml
```

## Enable Camel Subsystem

The patch does not modify existing configuration files. Instead it comes with a number of additional configurations files that end in `-camel.xml`.

If you should want to add the Camel subsystem to existing configurations you can run the following command.

```
$ bin/fuseconfig.sh --configs=camel --enable
Processing config for: camel
    Writing 'layers=fuse' to: .../wildfly-9.0.0.CR1/modules/layers.conf
    Enable camel configuration in: .../wildfly-9.0.0.CR1/standalone/configuration/standalone.xml
    Enable camel configuration in: .../wildfly-9.0.0.CR1/standalone/configuration/standalone-full.xml
    Enable camel configuration in: .../wildfly-9.0.0.CR1/standalone/configuration/standalone-full-ha.xml
    Enable camel configuration in: .../wildfly-9.0.0.CR1/standalone/configuration/standalone-ha.xml
    Enable camel configuration in: .../wildfly-9.0.0.CR1/domain/configuration/domain.xml
```

There are currently three triggers that can be used to enable Camel for a deployment

1. a deployment that contains a *-camel-context.xml descriptor
2. a deployment that contains a type annotated with @CamelAware
3. a deployment that contains a type annotated with @ContextName

## Docker Image

The easiest and most portable way to run WildFly-Camel is to use the [wildflyext/wildfly-camel](https://registry.hub.docker.com/u/wildflyext/wildfly-camel/) distribution.

```
$ docker run --rm -ti -p 8080:8080 -p 9990:9990 -e WILDFLY_MANAGEMENT_USER=admin -e WILDFLY_MANAGEMENT_PASSWORD=admin wildflyext/wildfly-camel
```

## Maven Archetypes

To get started with writing Camel JEE applications, there are two Maven archetypes that can generate either a Camel Spring XML or Camel CDI application.

To generate a Camel CDI application, run the following from the command line.

<pre><code>
mvn archetype:generate -DarchetypeGroupId=org.wildfly.camel.archetypes \
                       -DarchetypeArtifactId=wildfly-camel-archetype-cdi \
                       -DarchetypeVersion={{ book.version }} \
                       -DgroupId=com.mycompany \
                       -DartifactId=my-camel-cdi-application
</pre></code>

To generate a Camel Spring XML application, run the following from the command line.

<pre><code>
mvn archetype:generate -DarchetypeGroupId=org.wildfly.camel.archetypes \
                       -DarchetypeArtifactId=wildfly-camel-archetype-spring \
                       -DarchetypeVersion={{ book.version }} \
                       -DgroupId=com.mycompany \
                       -DartifactId=my-camel-spring-application
</pre></code>

Instructions on how to build, test and run the project can be found within the generated README files. 
