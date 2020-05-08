---
title: OSGi and JAX-RS - About time!
layout: postx
category: code
tags: [archive, java, osgi, jaxrs]
---

Having worked on a number of [Eclipse](http://www.eclipse.org/) client and server based products I am a huge 
[OSGi](http://www.osgi.org/) fan and while seeing it inside big companies developing big infrastructure is satisfying 
it's far more interesting to it being used and blogged in smaller and more personal projects. For a while REST services 
meant hand-rolled Servlet based implementations but [JAX-RS](http://jcp.org/en/jsr/detail?id=311) is a fantastic 
framework and it's annotation based model means it doesn't suffer from a lot of the Interface/Factory/Cruft bloat of a 
lot of Java frameworks. Specifically work now involves a lot of service built with JAX-RS and more specifically our 
server platform is commonly referred to as "Java + [Jetty](https://www.eclipse.org/jetty/) + [Jersey](https://eclipse-ee4j.github.io/jersey/)". 
So, now the trick has been getting JAX-RS services running in an OSGi based server and a quick half hour on Google will 
show (or until recently) that a number of people have tried and either failed or succeeded in a limited fashion. 
Recently however we get Neil Bartlett's [JAX-RS OSGi Extender](http://github.com/njbartlett/jaxrs-osgi-extender); 
a bundle that allows the standard Jersey bundles (yes, those jar files the jersey project delivers are OSGi enabled) to 
run in an OSGi container.

So, does it work as advertised? Well I started with the README file on Github and a working [Apache Felix](https://felix.apache.org/)
configuration. I tend to gravitate toward Felix just because the footprint tends to be smaller, I get most of the same 
features, the web console is nice and, well I am just used to it. 

Here's the bundle list I started with.

```
org.apache.felix.bundlerepository-1.4.2.jar
org.apache.felix.configadmin-1.2.4.jar
org.apache.felix.dependencymanager-2.0.1.jar
org.apache.felix.dependencymanager.shell-2.0.1.jar
org.apache.felix.eventadmin-1.0.0.jar
org.apache.felix.fileinstall-2.0.8.jar
org.apache.felix.http.api-2.0.4.jar
org.apache.felix.http.bundle-2.0.4.jar
org.apache.felix.http.jetty-2.0.4.jar
org.apache.felix.http.proxy-2.0.4.jar
org.apache.felix.http.whiteboard-2.0.4.jar
org.apache.felix.log-1.0.0.jar
org.apache.felix.metatype-1.0.4.jar
org.apache.felix.prefs-1.0.4.jar
org.apache.felix.scr-1.4.0.jar
org.apache.felix.shell-1.4.1.jar
org.apache.felix.shell.tui-1.4.1.jar
org.apache.felix.webconsole-2.0.4.jar
```

Then I added the bundles listed on Github, the Jersey core and server bundles and the extender bundle itself. I added 
my simple service and got it to work just as Neil outlined (you have to remember to add the `JAX-RS-Alias` and 
`JAX-RS-Classes` properties to the bundle manifest file).

```
jersey-core-1.4.jar
jersey-server-1.4.jar
name.njbartlett.osgi.jaxrsextender-0.0.0.jar
```

This works for services that return `text/plain` but the services I deal in nearly all consume and produce JSON and if 
I change the `@Produces` attribute to `application/json` you'll get a runtime exception when Jersey processes the 
request. So now you need to add a few more bundles, luckily they're all from the jersey site so you don't have to dig 
around for them, and you'll be set. There are a few here, basically there's a dependency chain such that we really want 
_jersey-json_, but that depends on _jackson_ which in turn depends on _jettison_.

```
jersey-json-1.4.jar
jackson-core-asl-1.5.5.jar
jackson-jaxrs-1.5.5.jar
jackson-mapper-asl-1.5.5.jar
jackson-xc-1.5.5.jar
jettison-1.1.jar
```

The final result is a nice clean service shown below, we define a specific class that is our domain object, return it 
from our API and define the content type and let Jersey do the rest.

```java
import javax.ws.rs.GET;
import javax.ws.rs.Path;
import javax.ws.rs.Produces;

@Path("/sysprop")
public class SampleService {
 
    @GET
    @Produces("application/json")
    public PlatformProperties getSystemProperties() {
        ...
    }
}
```

One final thing that tripped me up was that by default the JSON mappers included above don't process "unadorned" classes, they need a hint to know where to start the serialization so you need to add the "@XmlRootElement" attribute to the domain object.


```java
import javax.xml.bind.annotation.XmlRootElement;

@XmlRootElement
public class PlatformProperties {
    ...
```