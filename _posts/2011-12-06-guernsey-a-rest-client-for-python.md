---
title: Guernsey - a REST client for Python
layout: postx
category: code
tags: [archive, python, rest]
---

Having written a number of REST clients in Python over the last few years, they all start out the same way; write a 
simple piece of code to call one service, then realize that you have to call a second service, then a third and so the 
code becomes more and more generic. However, starting the next project I have to untangle that code from project A and 
copy into project B - if possible, changing an employer does make code reuse a pain.

So this time, when the need arose I decided to write the client as a separate and reusable package - 
[Guernsey](https://github.com/johnstonskj/guernsey). I then looked at the API I had developed before and it really 
wasn't very good, or rather it still showed aspects of the first project I created it for. Right now my day job also 
includes writing Java REST services and we use JAX-RS which give us a really annotation based model. As an 
implementation we use the [Jersey](https://eclipse-ee4j.github.io/jersey/) implementation which also has a very nice 
client package, so I set out to see if I could use that design as a basis for a Python package.

The result is pretty close to the Jersey API, which may be of value to anyone working in Java and Python (which I do 
regularly) but also it simply benefits from the effort someone else put into the API design. I also took care with 
naming to keep things as close as possible, but to use a Python style (lowerCamelCase became lower_snake_case, some 
getter/setters became simple object properties, etc.).

The following example shows a simple client that accesses a public REST service.

```python
from guernsey import Client
 
client = Client.create()
resource = client.resource('http://www.thomas-bayer.com/sqlrest/')
resource.accept('*/xml')
namespaces = resource.get()
```

1. The `Client` class is the entry point for most common cases, so for most uses you can simply import the one class.
1. Calling `Client.create` will construct a new `Client` object, this method takes a dictionary that provides 
   configuration parameters, or without configuration will create a new object with all default values.
1. Calling `client.resource` will construct a new `WebResource` object which is the actual object you perform REST 
   methods against.
1. The `WebResource` object has a number of methods which can be used to configure the resource behavior; in this 
   example we simply set the accept headers to denote that we wish to be returned XML data if possible.
1. Finally we call the get method on the `WebResource` (each `WebResource` supports `get`, `head`, `put`, `post`, 
   `delete` and `options` by default but could be extended to support other methods).

All the configuration methods on the return WebResource the object back to the caller which allows configuration methods 
to be chained together which can make some code more readable.

```python
namespaces = client.resource('http://www.thomas-bayer.com/sqlrest/') \
                   .accept('*/xml', 0.8) \
                   .accept('*/json', 0.2) \
                   .accept_language('en-US') \
                   .get()
```

In this example we can see that the setup methods `accept` and `accept_language` are all chained together and finally 
the `get` method is called. A number of simple methods are provided to manipulate common headers, although if you want 
to add your own header you can use the `add_header` method.

## Navigating Resources

Where your REST service follows a true hypertext (HATEOS) style one would expect that navigating between resources 
should be simple as any resource would provide within it a navigable URL from it to any related resource. However, some 
services still do not provide a true hypertext style, either returning relative URLs or worse returning identifiers 
which a client has to use to construct a URL.

Guernsey should help you in all three cases:

```python
namespaces = resource.get()
data = namespaces.parsed_entity
# Hypertext link:
new_resource = client.resource(data['resource_url'])
# Relative path:
new_resource = resource.path(data['resource_path'])
# A sub-resource identifier:
new_resource = resource.sub_resource(data['resource_id'])
# Just an identifier:
new_resource = client.resource('http://example.com/{resource_id}', data)
```

* In the first case we have, in our parsed response data, a full URL and so we can easily call the `client.resource` 
  method to construct a new `WebResource`.
* In the second case we have a new path relative to the path of the current resource, in this case we can use the 
  `resource.path` method which will construct a new absolute URL by resolving the provided path against the current URL.
* In the third case we only have, in our parsed response data, an identifier which corresponds to a path segment that 
  can be appended to the current resource path (commonly termed a sub-resource identifier).
* In the last case we need to construct an entirely new resource, but using the identifier in the parsed response data, 
  in this case we use a templated resource URL in the call to `client.resource`.

## Handling Entities

In the section above we mentioned the use of a _parsed entity_, the Guernsey client has the ability to detect a known 
resource representation and then convert from its serialized form returned from the REST service into a common Python 
form. In all cases any entity returned from the service will be put into the entity property of the response, if the 
`Content-Type` header in the response identifies a type that can be de-serialized then the response will contain a 
`parsed_entity` property.

```python
namespaces = resource.get()
raw_data = namespaces.entity
xml_data = namespaces.parsed_entity
```

The client performs this by using the `EntityReader` and `EntityWriter` classes in the `guernsey.entities` module. These 
classes define a simple interface for a subclass to determine whether a given type can be serialized, and if so to 
convert it to/from a reasonable Python form. Instances of these classes are added to the array property `entity_classes` 
on the `Client` object and will be called in order to process any request and response.

The `guernsey.entities` module contains reader/writer classes for XML and for JSON and these are added by default to the 
standard `Client`. The JSON reader will convert serialized JSON into Python dictionaries and standard types using the 
standard `json` module, the writer will serialize Python dictionaries and standard types into the JSON serialized form. 
The XML reader and writer convert between serialized XML and `ElementTree` objects from the `xml.etree.ElementTree` 
standard module.

Additional entity readers and writers can easily be constructed, and if added to the client will be used by any resource 
created by the client.

## Client Filters

An additional capability of Jersey, and therefore Guernsey, is the ability to write client-side request/response 
filters. These filters work much like Servlet filters in Java or WSGI middleware in Python in that they are chained 
together, the handler is given the current request and is able to modify the request before passing it to the next 
handler in the chain. The last handler actually performs the request, constructs a response object and returns it back 
down the chain. Each handler is then able to modify the response before returning it itself.

A number of common filters are provided to enable logging, Gzip encoding and MD5 content hashes. These can be added 
simply by using the `add_filter` method on a resource.

```python
resource = client.resource('http://www.thomas-bayer.com/sqlrest/')
namespaces.add_filter(LoggingFilter('TestFilterLogging'))
response = namespaces.accept('*/xml').get()
```

Each handler will have a `handle` method which will be given a client request and be expected to return a client 
response. Each handler will also be responsible for calling `client_request.next_filter` to ensure that all filters are 
chained correctly. For example, the following is the implementation of the handler method in the logging filter shown 
above.

```python
def handle(self, client_request):
    logger = logging.getLogger(self.log_name)
    try:
        logger.info(client_request.method + ' ' + client_request.url)
    except:
        print 'Error writing request to log'
    client_response = client_request.next_filter(self).handle(client_request)
    try:
        logger.info(client_response.url + ' ' +
            str(client_response.status)+ ' ' +
            client_response.reason_phrase)
    except:
        print 'Error writing response to log'
    return client_response
```

## Implementation

I chose to use `urllib2`, its low level, has a lot of features although not as many as Joe Gregorio's `httplib2`, but 
have also a lot of experience using it. I may well change to using `httplib2` in the future and there's no reason the 
Guernsey API should change whichever underlying module I use.

On the name, if you haven't figured it out yet, both Jersey and Guernsey are part of the Channel Islands.