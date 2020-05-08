---
title: Some OSGi JAX-RS tweaking
layout: postx
category: code
tags: [archive, java, osgi, jaxrs]
---

In my last post I managed to get my Felix OSGi run time to host some JAX-RS resources, but noted one issue then and had 
one issue since that were worth a little effort to understand.

## Nasty JAX-B attributes

In the last post I noted that to get my data to serialize correctly I had to add JAX-B attributes to my domain classes 
to ensure that the objects would be correctly serialized.

```java
import javax.xml.bind.annotation.XmlRootElement;

@XmlRootElement
public class PlatformProperties {
    ...
```

First, I only ever product JSON so the whole "Xml" prefix for each attribute is kind of annoying, and secondly I'd just 
rather not have to provide any attributes at all if the runtime can figure out a sensible serialization for me. Now, 
the JAX-RS runtime has a provision for custom serialization, the notion of a `MessageBodyWriter` which is then added to 
the list of classes for an application. The Jackson project does provide a nice implementation that can read/write JSON 
and doesn't need all those attributes - `org.codehaus.jackson.jaxrs.JacksonJsonProvider`.

The issue is that the current implementation of the OSGi Extender takes a list of classes from the bundle manifest to 
create the application. Because the list of classes provided by the bundle the extender uses the bundle class loader to 
create the actual list of classes to provide to Jersey. Unfortunately the JSON provider isn't in my bundle so we can't 
list it in the `JAX-RS-Classes` manifest property as the class loader will fail. To get around this, I created the 
following simple subclass in my own bundle.

```java
import org.codehaus.jackson.jaxrs.JacksonJsonProvider;

public class JsonProvider extends JacksonJsonProvider {

}
```

I then added this to the list of classes in `JAX-RS-Classes` and was able to serialize my domain classes nicely - sans 
attributes.

## OSGi service can't be a resource

My next issue was that I had a nice OSGi service (in fact using declarative services) and some of it's methods would be 
nice to expose to turn the service effectively into a resource. So, starting with the obvious I just added my JAX-RS 
attributes to the implementation class for my service - which almost worked. My service started, and like a good citizen 
I had used a declarative service reference to inject the `LogService` into my service. However, when I used my browser 
to pull data from my resource I was rather surprised to get a `NullPointerException` as my service didn't even do 
anything yet, here it is.

```java
@Path("/things")
public class MyServiceImpl implements MyService {

    private LogService log = null;

    @Override
    @GET
    @Produces("application/json")
    public List getIds() {
        log.log(LogService.LOG_INFO, "Retrieving list of IDs.");
        List idList = new LinkedList();
        return idList;
    }

    public void bindLog(LogService log)  {
        this.log = log;
    }
 
    public void unbindLog(LogService log) {
        this.log = null;
    }
}
```

It turns out the NPE was the first line of the `getIds` method, the call to the log service. Basically the OSGi runtime 
created a singleton object for the service and bound an instance of the log to my service correctly. However, because 
the default behavior for the OSGi extender is to pass in a list of classes to Jersey via the `Application` class then 
Jersey will construct a new copy of `MyServiceImpl` as it needs to, one that hasn't been provided with a log service so 
the log is `null` and ... boom.

So, now I have to separate out my resource from my service, a shame but not necessarily a problem. Now, it would be 
possible to take the instance of my service and pass it to Jersey as a singleton, but would make the implementation of 
the extender much more complex.
