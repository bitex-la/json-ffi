# json-ffi

Use a jsonapi.org compliant API over FFI. 

# A JSON-FFI producer

A JSON-FFI producer is analogous to a JSON-API server, here's how it translates.

## Local shared libraries instead of web servers.

You MUST provide files named with your library name, prefixed by the word 'lib'.
The file extension for your library MAY change as required by each platform (.dylib, .so, .dll)
Library names must be camel_cased #TODO: Reference the camelcase spec.

For example, if a library is commonly known as MemoryBlogger, the shared objects MUST be named:

libmemory_blogger.so, libmemory_blogger.dylib, libmemory_blogger.dll

## Exported FFI handler function

- The producer MUST export a *handler function* via FFI.
- The *handler function* MUST receives a single C string, containing a *JSON-FFI Request*.
- The *handler function* SHALL return a *response C String* containing a *JSON-API Document*.
- The producer MUST export a freer function for the client to release a *response C string* from memory.
- The *freer function* SHALL receive a pointer to a *response C string* and MAY be idempotent.
- The *handler function* name MUST be "json_ffi_handler", prefixed by the library name.  (memory_blogger_json_ffi_handler)
- The *freer function* name SHALL be "json_ffi_freer", prefixed by the library name.  (memory_blogger_json_ffi_freer)

# A JSON-FFI consumer

A JSON-FFI consumer is analogous to a JSON-API client, here's how it translates.

## FFI local bindings instead of HTTP clients.

- A JSON-FFI consumer SHALL use FFI bindings to call into the shared library's *handler function* passing a C String containing a JSON-FFI Request as the only argument.
- The JSON-FFI consumer MUST call the library's *freer function* to release the response when appropiate.

# The JSON-FFI Request

A request to a JSON-API server has an http method, a url that may include a resource id and a
relationships link, and query parameters.

A request to a JSON-FFI producer provides the same information but using in a JSON object.

A JSON-FFI Request:

- MUST have a *method* member, equivalent to the HTTP method in JSON-API.
- MUST have a *type* member. Analog to the resource name at the beggining of a JSON-API URL.
  (http://jsonapi.org/format/#fetching-resources)
- MAY have an *id* member. Analog to the resource ID in a JSON-API URL.
  (http://jsonapi.org/format/#fetching-resources)
- MAY have a *relationship* member. Analog to the relationships part of a JSON-API request URL.
  (http://jsonapi.org/format/#fetching-relationships)
- MAY have an *include* member. Analog to the "include" query parameter in a JSON-API request
  (http://jsonapi.org/format/#fetching-includes)
- MAY have a *sort* member. Analog to the "sort" query parameter in a JSON-API request.
  (http://jsonapi.org/format/#fetching-sorting)
- MAY have a *fieds* member. Analog to the "fields" query parameter in a JSON-API request.
  (http://jsonapi.org/format/#fetching-sparse-fieldsets)
- MAY have a *page* member. Analog to the "page" query parameter in a JSON-API request.
  http://jsonapi.org/format/#fetching-pagination
- MAY have a *filter* member. Analog to the "filter" query parameter in a JSON-API request.
  http://jsonapi.org/format/#fetching-filtering
- MAY have a *data* member containing a resource object to create or update.
  (http://jsonapi.org/format/#crud-creating)


## Example

Given a standard HTTP JSON-API blog, where articles have authors and comments.

### To list all articles

To list articles
```
GET    /articles/               : To list all Articles.

```

### To list all articles, paginated, sorted, filtered, and including comments.

### To create a new article
```
POST   /articles/               : To create a new Article
```

### To perform a custom collection action (get stats about all your articles)
```
GET    /articles/stats          : A custom collection action (gets stats about all your Articles).
```

### To show a single Article
```
GET    /articles/101     : To show a single Article with id 101
```

### To show a single article, including author and comments.

### To 
PATCH  /articles/101     : To edit article 101 (as if you could).
POST   /articles/101/pin : A custom member action (pins Article 101).
DELETE /articles/101     : To delete Article 101.

# What's a client?


# Goals and Phylosohpy

## Help polyglot teams, keeping distribution and deployment simple.

Lots of Python, PHP, Ruby, or JS projects want to use some Rust, Go, C, C++, Haskell.
The reason may be type safety, memory optimizations, algorithm execution speed, developer availability, or just personal preference.

With JSON-FFI you could write a producer in Rust and share it with your Python or Ruby teams via a
binary distribution Ruby Gem or Python Egg. They can use your library much like they consume web services.

You can have Travis.ci build your Linux and Mac binaries too.

## Stop Bikeshedding about all API's.

Thanks to web API's and microservice architectures we've learned a lot about public API's.
- Schemas are nice, but succesful API's rely primarily on documentation.
- Almost anything can be represented as a resource.
- Human readibility, familiarity and community support are very important.

So whenever we start writing an API to be used on a network we usually default to HTTP & Rest & JSON.

Yet communicating with a native library is about thinking and writing FFI code (or schemas that generate it) each time.

With JSON-FFI you can stop bikeshedding in your native library API's.
And if your library ends up being a webservice, user's wont have to change much.
(at least in all those cases where performance tuning is not part of the job) 

## Reuse knowledge

How many implementations of the credit card validation algorithm do we need?

We have it for [python](https://github.com/orokusaki/pycard),
[ruby](https://github.com/Fivell/credit_card_validations),
[javascript](https://github.com/braintree/card-validator),
[php](https://github.com/inacho/php-credit-card-validator)
and if your language is not on the list, you can choose to use a
[webservice](https://www.bincodes.com/api-creditcard-checker/)

This duplication makes sense for a number of reasons: Often times a rewrite is more convenient than depending on another
language's runtime to be available, and if a native library was available, writing FFI bindings is still quite unfamiliar.

If using a native library was similar to consuming a webservice we would be reusing more.

## Favor collaboration and community over serialization speed.

If you agree that

> "Programs must be written for people to read, and only incidentally for machines to execute." -- Hal Abelson, 1985.

Then you may also agree that

> "API's are for programmers to talk, and only incidentally for machines to communicate." -- Anonymous, just now.

JSON was chosen as because it has the largest community of users.

There are [binary formats](https://en.wikipedia.org/wiki/Comparison_of_data_serialization_formats#endnote_pbtextformat)
focused on smaller message sizes and faster parsing, but they are harder to explore and debug,
and in some cases (like google's protobuf) are very invasive in the developer's toolchain.

The performance gain from binary formats seems to be less than 7X, which puts the tradeoff in
perspective. Find some benchmarks and comparisons
[here](http://blog.celogeek.com/201401/519/perl-benchmark-jsonxs-vs-sereal-vs-datamessagepack/),
[here](https://jsperf.com/msgpack-js-vs-json/37) and
[here](https://auth0.com/blog/beating-json-performance-with-protobuf/).

The [JSON-API spec](http://jsonapi.org) was chosen because of it's anti-bikeshedding philosophy,
active community, and because the resource paradigm can express a wide range of applications.

