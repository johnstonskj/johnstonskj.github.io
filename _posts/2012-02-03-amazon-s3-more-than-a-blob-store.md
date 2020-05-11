---
title: Amazon S3, more than a blob store
layout: postx
category: code
tags: [archive, aws]
---

Having built a number of RESTful services I have developed the storage for resources a number of times a number of 
different ways. It's important to realize that there is a progression that the first few projects go through until you 
work out the patterns that really work. The easy ones are obvious, keep the HTTP methods simple, avoid adding crazy 
request/response headers, use ETags, if you need content negotiation only use the standard Accept/Content-Type 
mechanism. The fun starts when you try and wrap your head around updates to resources, and the easiest way to get 
yourself out of the spiral discussion of partial updates, support for PATCH and all manner of other craziness is to 
treat all resources as technically immutable and every PUT to the resource creates a new version of the resource. This 
works well as the version identifier becomes a very stable ETag and with a little effort you can expose a URI for each 
version allowing you to look back and forward through the history of a resource. For a lot of projects I've used the 
resource store to store temporary resources (sometimes as a queue, the output from process one is put into the store to 
be picked up by process two) and it's annoying to keep having to write another "reaper" process to remove expired 
resources.

Recent changes to the Amazon S3 service have really made this a viable resource store, it's not just an online file 
store any more. While S3 has supported metadata associated with objects and a great API for query it's the additions to 
the API recently that

* Versioning, it's been there a while actually but not many people use it. Basically you can turn versioning on 
  bucket-by-bucket to store all versions of every object in the bucket. The query API is now extended so as well as 
  listing all objects in a bucket you can now list all the versions of an object.
* Expiration, you can now specify an expiration time for an object and S3 will automatically remove the object. 
* Multi-Delete, if you have to still perform your own clean-up operation you can now query for all "expired" or 
  otherwise superfluous resources and delete a set of resources in a single API call to S3.

This makes S3 a much more complete solution and certainly I hope I don't have to write another RESTful resource store 
any time soon, at least anything more than a domain wrapper around S3.
