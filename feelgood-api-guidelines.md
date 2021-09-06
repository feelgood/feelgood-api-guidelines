# Guidelines for Feelgood's HTTP API:s

These are pragmatic and practical guidelines for Feelgood's HTTP API:s.

It is more difficult to design and provide a public API than an internal API. These guidelines were originally written for Feelgoods internal API:s, but are applicable to external API:s.

See the last section **References** in this document for suggested reading material.

REST is an architectural style. HTTP is a application-level protocol. Understand that!

These Guidelines are inspired by [Zalandos HTTP API Guidelines](https://github.com/zalando/restful-api-guidelines).

## Priciples

 * Imagine that the API could eventually be made **public**.
 * Use web standards where they make **sense**.
 * Be **friendly** to the developer and be explorable via a browser address bar.
 * Be **simple**, **intuitive** and **consistent** to make it easy and pleasant to use.
 * Be **pragmatic** and **practical** so we don't over-engineer or procrastinate.

## The Guidelines

### MUST: Document the API

Use [The Open API Specification](https://spec.openapis.org/oas/latest.html) and/or [Swagger specification](https://swagger.io/specification). Commit the specfication file to the github repository for the API.

### SHOULD: The API specification file should be committed to the doc directory of the API:s git repo

Writing YAML is nicer than JSON.

For example:
```
./doc/openapi.yml
```

### MUST: Use US english in the API

For consistency. US spelling is chosen because it in some cases means slightly fewer characters! :-) Example: **colour** (British) and **color** (US). **traveller** (British) and **traveler** (US).

See article in [Oxford Living English Dictionaries](https://en.oxforddictionaries.com/spelling/british-and-spelling).

### MUST: Use HTTPS

All installations of API:s that have production-like data and are publicly exposed must use HTTPS and proper server certificates.

### SHOULD: Avoid versioning

Avoid versioning the API. It is the media types that should be versioned.

See what [Roy Fielding said about versioning](https://www.infoq.com/articles/roy-fielding-on-versioning).

> Websites don’t come with version numbers attached because they never need to.
> Neither should a RESTful API. ... What you are creating is not a new version of
> the API, but a new system with a new brand. -- Roy Fielding.

[This](http://www.baeldung.com/rest-versioning) is also a pretty good (and short) blog article on the challenges of versioning HTTP API:s.

### SHOULD: Use Media Type Versioning

In this example, a client wants only version 2 of the response:

```
Accept: application/x.feelgood.organization+json;version=2
```

A server responding to this, as well as a client sending a request with content should use the Content-Type header, declaring that one is sending the new version:

```
Content-Type: application/x.feelgood.organization+json;version=2
```

### MUST: Use JSON objects

Use JSON for HTTP body contents and use the mediatype **application/json**.

**Note**: Always use JSON objects. Also for collections. It is easier to extend the representation with meta-attributes later.

### MUST: Design the API as a set of resources

A REST-ful HTTP API provides **resources** and a set of standardized methods for manipulating them; GET, PUT, POST etc. When designing an API you must think about which business concepts and information you want to handle. These concepts should be modeled as resources and it is these resources you want to expose in the API. Finally you need to think about how you want to represent the resources as data (media types).

A very simple classification of resources is **informational resources** and **transactional resources**.

Informational resources are typically made persistent via data files, documents in document databases or rows in relational databases. Informational resources can also be session-based and kept in a memory-only data store like memcached. An example of a classic session-based resource is a web shopping cart and its items.

Transactional resources typically represent operations or processes that can be started, ongoing, paused, stopped and completed. They can have data associated with them that is made persistent for the duration of the transaction. Examples of transaction resources are a) a remote build of a program in a CI-system and b) a transfer of money between bank accounts. Transaction resources may be made to only exist for the duration of the transaction, but sometimes it may be interesting to keep resources representing completed transactions.

Since both informational and transactional resources will have data representations it is highly recommended that these resources are named as **nouns**.

### MUST: Use human readable names in URL:s

There is no formal requirement or reason for URL:s and the resources they are used to identify, to have human readable names. But, it makes an API much easier to understand and document if human readable names are used. So do that.

Use names that clearly communicate the semantics of the concepts that the resources represent.

### MUST: Use plural forms for resources represented by collections

### MUST: Use snake case for resource names and query parameters

Do not use CamelCase!

**Comment**: But Camelcase is the default in the Javascript code that typically will consume the API. Yes, but API:s can be consumed by programs and systems written in other languages.

### SHOULD: Use standardized HTTP Headers

### MUST Non-standard HTTP Headers MUST be prefixed with **X-**.

An example:
```
X-very-interesting-stuff: 23 quarks
```

### MUST: Use standardized HTTP Status Codes

### SHOULD: Use conventional query string attribute names

Query string attributes should support **paging**, **filtering**, **sorting**, **projection** for resources representing collections of items.

Paging solves the need to be able to request and return subsets of a large collection. The following query attributes should be used:

 * **limit=n**. n is the maximum number of items to return (from offset). The default should be some reasonable value like 10 or 100.
 * **index=i**. i is the starting index for items. The default is 0.

Filtering solves the need to be able to request and return a subset of a large collection based on item attribute values. The following query attribute should be used:

 * **filter[attributename]=valuespec**. attributename is the name of the attribute to filter by and valuespec that value of the attribute that items must have to be returned.

Sorting solves the need to be able to request and return a collection sorted alphanumerically on some item attribute value. The following query attribute should be used:

 * **sort[attributename]=forward|reverse**. attributename is the name of the attribute to sort by. The value forward (default) and reverese specifies the sorting directoion.

Projection solves the need to be able to request and return partial JSON objects for all items in a collection. The following query attribute should be used:

 * **attributes=attributenames**. attributenames is a comma-separated list of attribute names for those attributes that are to be returned in each JSON object.

### SHOULD: Items in collections should contain a "link" attribute with the URL of the item as its value

This must be implemented if the collection resource supports projection.

An example:

In a collection of 5000 items representing companies the item representing company 1684 may only have its name and a link in the JSON object representing it:
```
{"name": "Foobar AB"
 "link": {"href": "/companies/1684", "rel": "item"}}
```

Since the projection request only requested company names to be included in the response, the client must be able to easily request more information on a given company.

### MAY: Implement HATEOAS-ful links in all resource representations

This is tough and non trivial.

### MAY: Use the HTTP Link Header for hyperlinks

See page 6 of [_Web Linking_, IETF RFC 5988 (Proposed Standard)](https://tools.ietf.org/html/rfc5988#page-6).

### MAY: Subset representations may have links to next, previous, first and last.

The JSON object representing a subset of a collection may have link attributes for the next, previous, first and last subset (page).

### SHOULD: Date property values should conform to RFC 3399

See [_Date and Time on the Internet: Timestamps_, IETF RFC 3399](https://tools.ietf.org/html/rfc3339).

### SHOULD: Standards should be used for Language, Country and Currency

For **countries** use code sets 3166-1 or 3166-2 of [_Codes for the representation of names of countries and their subdivisions_, ISO 3166](https://en.wikipedia.org/wiki/ISO_3166)

For **languages** use [_Tags for Identifying Languages_, IETF RFC 5646](https://tools.ietf.org/html/rfc5646) and [_Language codes_, ISO 639](https://www.iso.org/iso-639-language-codes.html).

For **currencies** use [_Currency codes_, ISO 4217](https://www.iso.org/iso-4217-currency-codes.html).

### MUST: Use API keys for external systems that need to use the API

By external systems are meant partners' and customers' systems.

### MUST: Use user authentication tokens when accessing user's resources and data

### MAY: Use the Server-Timing header to provide timing info to clients

Use the W3C header **Server-Timing** to provide timing info to clients.

Two examples:
```
Server-Timing: BFF-API;dur=666, some-backend-api;dur=87, some-backend-mongo-db;dur=54

Server-Timing: BFF-API;dur=1034, another-backend-api;dur=898, some-backend-SQL-db;dur=874
```

Providing this information to clients has security issues, since you may reveal information on your backend API:s, systems and databases. One solution to this is to only provide the header when the client also provides a key or password. This can be easily provided in HTTP client scripts or tools like Postman.

Be aware that providing this information, incurs a slight performance price, since the server API needs to track the execution time and add the header.

For more info, see:

 * [Server Timing: W3C Working Draft 30 June 2021](https://www.w3.org/TR/server-timing/)
 * [Supercharging Server Timing with HTTP trailers](https://www.fastly.com/blog/supercharging-server-timing-http-trailers)
 * [Mozilla Web Docs: Server-Timing](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Server-Timing)
 * [Measuring Performance With Server Timing](https://www.smashingmagazine.com/2018/10/performance-server-timing/)

## References

### REST

 1. Chapter 5 on the REST Architectural Style in [_Architectural Styles and the Design of Network-based Software Architectures_](https://www.ics.uci.edu/%7Efielding/pubs/dissertation/rest_arch_style.htm). Roy Fielding's doctoral dissertation (2000).
 2. [_RESTful Web Services_](https://www.safaribooksonline.com/library/view/restful-web-services/9780596529260/). by Sam Ruby & Leonard Richardson. Good book on REST and HTTP and the notion of a resource-oriented architecture.
 3. [_RESTful Web APIs_](https://www.safaribooksonline.com/library/view/restful-web-apis/9781449359713/). by Sam Ruby, Mike Amundsen & Leonard Richardson. Good book on REST and REST-ful HTTP API:s.

### HTTP

 4. [_Hypertext Transfer Protocol (HTTP/1.1): Message Syntax and Routing_, IETF RFC 7230](https://tools.ietf.org/html/rfc7230)
 5. [_Hypertext Transfer Protocol (HTTP/1.1): Semantics and Content_, IETF RFC 7231](https://tools.ietf.org/html/rfc7231)
 6. [_Hypertext Transfer Protocol (HTTP/1.1): Conditional Requests_, IETF RFC 7232](https://tools.ietf.org/html/rfc7232)
 7. [_Hypertext Transfer Protocol (HTTP/1.1): Range Requests_, IETF RFC 7233](https://tools.ietf.org/html/rfc7233)
 8. [_Hypertext Transfer Protocol (HTTP/1.1): Caching_, IETF RFC 7234](https://tools.ietf.org/html/rfc7234)
 9. [_Hypertext Transfer Protocol (HTTP/1.1): Authentication_, IETF RFC 7235](https://tools.ietf.org/html/rfc7235)

### JSON

 10. [_The JavaScript Object Notation (JSON) Data Interchange Format_](https://tools.ietf.org/html/rfc7159)
 11. [_Introducing JSON_](http://json.org/). Jason Crockford's gentle and friendly description of JSON.

### Other REST and HTTP API guidelines

 12. [_A specification for building APIs in JSON_](http://jsonapi.org/).
 13. [_Zalando RESTful API and Event Scheme Guidelines_](http://zalando.github.io/restful-api-guidelines/)
 14. [_Best Practices for Designing a Pragmatic RESTful API_](http://www.vinaysahni.com/best-practices-for-a-pragmatic-restful-api).
