- Start Date: 2016-07-01
- Version: 0.8.0-DRAFT

# Summary

An opinionated guide to designing, structuring and maintaining microservice
RESTful HTTP APIs.

# Table of Contents

<!-- TOC depthFrom:1 depthTo:3 withLinks:1 updateOnSave:1 orderedList:0 -->

- [Summary](#summary)
- [Table of Contents](#table-of-contents)
- [Motivation](#motivation)
- [Detailed Design](#detailed-design)
  - [Vocabulary](#vocabulary)
  - [Scope](#scope)
    - [Existing Services](#existing-services)
  - [Design Principles](#design-principles)
  - [Archetypes](#archetypes)
    - [Docroot](#docroot)
    - [Resource](#resource)
    - [Collection](#collection)
    - [Controller](#controller)
  - [Naming Conventions](#naming-conventions)
  - [URL Structure](#url-structure)
    - [URLs for Resource Collections](#urls-for-resource-collections)
    - [URLs for Individual Resources](#urls-for-individual-resources)
    - [Relationship URLs and Related Resource URLs](#relationship-urls-and-related-resource-urls)
    - [Canonical Identifier](#canonical-identifier)
    - [URL Controllers and Verbs](#url-controllers-and-verbs)
    - [URL Length](#url-length)
  - [Methods](#methods)
    - [DELETE](#delete)
    - [GET](#get)
    - [OPTIONS](#options)
    - [PATCH](#patch)
    - [POST](#post)
    - [PUT](#put)
  - [Request Headers](#request-headers)
  - [Response Headers](#response-headers)
  - [Custom Headers](#custom-headers)
  - [Response Formatting](#response-formatting)
    - [Golden Rules](#golden-rules)
    - [Basics](#basics)
    - [Date Formatting](#date-formatting)
    - [JSON API Specification](#json-api-specification)
    - [Booleans, NULLs and Stringification](#booleans-nulls-and-stringification)
  - [Filtering](#filtering)
  - [Pagination](#pagination)
  - [Sorting](#sorting)
  - [Kong](#kong)
  - [Monitoring](#monitoring)
    - [Health](#health)
    - [Debug](#debug)
    - [Metrics](#metrics)
    - [Status](#status)
    - [Version](#version)
  - [Schema Documentation](#schema-documentation)
  - [Asynchronous Processing and Long-Running Operations](#asynchronous-processing-and-long-running-operations)
  - [Push Notifications](#push-notifications)
  - [Testing.](#testing)
  - [Versioning](#versioning)
    - [Breaking Change](#breaking-change)
    - [Deprecation](#deprecation)
    - [How and When to Version](#how-and-when-to-version)
  - [Security](#security)
    - [CORS](#cors)
    - [PII Parameters](#pii-parameters)
    - [Secure Connections](#secure-connections)
  - [Taxonomy](#taxonomy)
    - [Errors](#errors)
    - [Faults](#faults)
  - [Metrics](#metrics)
    - [Availability](#availability)
    - [Latency](#latency)
    - [Time to Complete](#time-to-complete)
    - [Long Running API Faults](#long-running-api-faults)
  - [Client Guidelines](#client-guidelines)
    - [Ignore Rule](#ignore-rule)
    - [Silent Failures](#silent-failures)
    - [Variable Ordering](#variable-ordering)
  - [Miscellaneous](#miscellaneous)
  - [Reference APIs and Libraries](#reference-apis-and-libraries)
    - [APIs](#apis)
    - [Libraries](#libraries)
  - [FAQ](#faq)
  - [See Also](#see-also)
  - [References](#references)
- [Drawbacks](#drawbacks)
- [Alternatives](#alternatives)
  - [Impact of Not Standardizing](#impact-of-not-standardizing)
- [Unresolved Questions](#unresolved-questions)

<!-- /TOC -->

# Motivation

The purpose of this RFC is to provide guidance on building microservice RESTful
HTTP APIs by detailing sane best practice conventions, more importantly, why
we should use them.

These conventions are not made up. They are sourced primarily from open source
and community standards, and most importantly, should be authored, owned and
the developers who use them.

If conventions are not managed carefully and implemented consistently,
integration and maintenance costs can grow exponentially as microservices
proliferate. Standards can and should be leveraged to mitigate this overhead by
setting expectations for all new services ahead of time, both intuitively and
programmatically.

Our goal is to set forth a set of sensible, lightweight conventions and
standards which facilitate development and integration of microservices. If
implemented correctly, these guidelines should make microservice development
**faster** and attainment of company goals **easier**.

# Detailed Design

This document establishes the guidelines all RESTful HTTP APIs should follow so
these RESTful interfaces are developed consistently.

These days, developers access most resources via HTTP interfaces. Although some
services provides language-specific frameworks to wrap API calls, almost all
operations eventually boil down to HTTP requests. Developers must support a wide
range of clients and services, and we cannot assume on rich frameworks are
available for every API consumer and development environment. Thus, the goal of
these guidelines is to ensure RESTful HTTP APIs can be easily and consistently
consumed by any client with basic HTTP support.

To provide the smoothest possible experience for developers, it's important to
have our APIs follow consistent design guidelines, rendering them intuitive and
easy-to-use.

The benefits of consistency accrue in aggregate as well; consistency allows
teams to leverage common code, patterns, documentation and design decisions.

These guidelines aim to achieve the following:

* Define consistent practices and patterns for all API endpoints.
* Adhere as closely as possible to accepted HTTP/REST best practices in the
  developer community at-large.
* Make accessing services via REST interfaces easy and intuitive for all
  application developers and API consumers.
* Allow service developers to leverage and reuse prior work on other services
  to implement, test and document consistently defined RESTful endpoints.
* Enable partners (e.g. non-developers, third parties) to use these guidelines
  as a playbook for building API-consuming applications and integrations.

## Vocabulary

Each guideline describes either a good or bad practice, and all have a
consistent presentation.

The wording of each guideline indicates the strength of the recommendation.

> **Do** is one that should always be followed. Always might be a bit too strong
> of a word. Guidelines that literally should always be followed are extremely
> rare. On the other hand, we need a really unusual case for breaking a Do
> guideline.

> **Consider** guidelines should generally be followed. If you fully understand
> the meaning behind the guideline and have a good reason to deviate, then do
> so. Please strive to be consistent.

> **Avoid** indicates something we should almost never do.

The keywords "MUST," "MUST NOT," "REQUIRED," "SHALL," "SHALL NOT," "SHOULD,"
"SHOULD NOT," "RECOMMENDED," "MAY," and "OPTIONAL" in this document are to be
interpreted as described in [IETF RFC 2119][rfc-2119].

## Scope

These guidelines are applicable to any RESTful HTTP API exposed publicly.
Private and internal APIs SHOULD also follow these guidelines, not least because
internal services are quite often exposed publicly later in their life cycles.
Consistency is valuable not only to external customers and front end
applications, but also internal service consumers, and these guidelines offer
best practices useful for any service.

There are legitimate reasons for exemptions to these guidelines. A REST service
which implements or must interoperate with some externally defined API must be
compatible with that API, and not necessarily with these guidelines. Some
services MAY also have special performance needs that require a different
format, such as a binary protocol.

### Existing Services

Services which pre-date these guidelines SHOULD become compliant at the next
version release which incorporates a breaking change. Respective service owners
should develop a strategy for bringing their services within compliance at or
before the service's next major release.

Dormant or deprecated services SHOULD become compliant if their use is forecast
to continue more than 12 months following the ratification of these guidelines.

## Design Principles

If we are ever in doubt on an API design or implementation choice, may these
Design Principles help guide us to the correct decision.

1. **Build for the consumer.**  
   &nbsp;  
   APIs do not exist for their own sake, and are almost never the entry point
   for end users. Be mindful of which applications will be consuming our
   services, and go out of your way to cater to their requirements.  
   &nbsp;  
2. **Convention over configuration.**  
   &nbsp;  
   Our APIs should be intuitive, and should adhere to and implement community
   standards wherever possible. Problem-solving is at the core of development,
   and is also the great expense. Avoid implementing custom solutions to
   problems others have already solved, and in doing so we'll avoid creating
   new problems of our own design for API consumers and code base maintainers.  
   &nbsp;  
3. **Optimize close to the metal.**  
   &nbsp;  
   From authorization and serialization to filtering and sorting, we will gain
   greater efficiencies the closer we are to the database. Keep this in mind
   when building multi-layer and service-dependent APIs.

## Archetypes

When modeling an API’s resources, we can start with the some basic resource
archetypes. Like design patterns, the resource archetypes help us consistently
communicate the structures and behaviors that are commonly found in REST API
designs. A REST API is composed of four distinct resource archetypes: _docroot_,
_resource_, _collection_, and _controller_.

Each of these resource archetypes is described in the subsections that follow.

### Docroot

A REST API’s root resource is commonly referred to as a _docroot_, also known as
a "reference document", and is typically the advertised entry point to an API.

The example URI below identifies the docroot for a contrived Soccer REST API:

```
https://api.soccer.restapi.org
```

When determining an API’s URL structure, it is helpful to consider that all of
its resources exist in a single reference document in which each resource is
addressable at a unique path. Resources are grouped by type at the top level of
this document. Individual resources are keyed by ID within these typed
collections. Attributes and links within individual resources are uniquely
addressable according to the resource object structure described above.

This concept of a reference document is used to determine appropriate URLs for
resources as well as their relationships. It is important to understand that
this reference document differs slightly in structure from documents used to
transport resources due to different goals and constraints. For instance,
collections in the reference document are represented as sets because members
must be addressable by ID, while collections are represented as arrays in
transport documents because order is significant.

### Resource

A resource is a singular concept that is akin to an object instance or database
record. A resource's state representation typically includes both _fields_ with
values and _links_ to other related resources. With its fundamental field and
link-based structure, the resource type is the conceptual _base archetype_ of
the other resource archetypes. In other words, the three other resource
archetypes can be viewed as specializations of the resource archetype.

Each URI below identifies a resource:

```
https://api.soccer.restapi.org/alerts/245743
https://api.soccer.restapi.org/leagues/seattle
https://api.soccer.restapi.org/users/1234
```

A document may have child resources, also known as nested resources, which
represent its specific subordinate concepts.

Each URI below identifies a nested resource.

```
https://api.soccer.restapi.org/leagues/seattle/sponsors/microsoft
https://api.soccer.restapi.org/leagues/seattle/teams/140
https://api.soccer.restapi.org/users/1234/favorites/alonso
```

### Collection

A collection is a server-managed directory of resources. Clients may propose new
resources to be added to a collection. However, it is up to the collection to
choose to create a new resource, or not. A collection resource chooses what it
wants to contain and also decides the URIs of each contained resource.
Collections may also be nested.

Each URI below identifies a collection resource:

```
https://api.soccer.restapi.org/leagues
https://api.soccer.restapi.org/leagues/seattle/sponsors
https://api.soccer.restapi.org/leagues/11/teams
```

### Controller

A controller models a procedural concept. Controllers are like executable
functions, with parameters and return values; inputs and outputs.

Like a traditional web application’s use of HTML forms, a REST API relies on
controllers to perform application-specific actions that cannot be logically
mapped to one of the standard methods (create, retrieve, update, and delete,
commonly referred to as "CRUD"). See the [Methods section](#methods) below for
more details.

Controller names typically appear as the last segment in a URI path, with no
child resources to follow them in the hierarchy. The example below shows a
controller resource that allows a client to resend an alert to a user:

```http
POST /alerts/245743/resend
```

_**Note:** Controller endpoints are exceptionally rare in modern RESTful APIs.
If we find ourselves implementing more than a couple of controller endpoints,
it may be worth rethinking our API design._

## Naming Conventions

> **Do** start and end member names with the characters "a-z" (U+0061 to
> U+007A).

> **Do** limit member names to the characters "a-z" (U+0061 to U+007A), "0-9"
(U+0030 to U+0039), and hyphen minus (U+002D HYPHEN-MINUS, “-“) as separator
between multiple words.

Allowed and recommended characters for URL-safe naming of members, response
objects and attributes are defined as part of the
[JSON API Specification](#json-api-specification).

<http://jsonapi.org/format/#document-member-names>

## URL Structure

> **Do** build API URLs which are easy for humans to read and for applications
> to construct.

> **Do** make resource names nouns with easy, conventional plurals (i.e. add an
> "s").

> **Avoid** nesting child resource relationships more than once. Instead,
> prefer to locate resources at the root path.

Human-readable URL structures facilitate discovery and ease cross-platform
adoption without requiring a heavyweight or language-specific client library.

An example of a well-structured URL is:

```
https://api.example.com/people/jdoe@contoso.com/assets
```

An example URL that is not friendly is:

```
https://api.example.com/EWS/OData/Users('jdoe@contoso.com')/Folders('AAMkADdiYzI1MjUxAAAAAACzMsPHYH6HQoSwfdpDx-2bAAA=')
```

Using URLs as parameters is a common design pattern. Services MAY use URLs as
values. For example, the following is acceptable:

```
https://api.example.com/images?url=https://resources.example.com/shoes/fancy
```

In data models with nested parent/child resource relationships, paths could
become deeply nested if you're not careful. Related collections should be nested
_at most_ one level deep.

**Don't do this.**

```
/orgs/{org_id}/apps/{app_id}/dynos/{dyno_id}
```

Limit nesting depth by preferring to locate resources at the root path. Use
nesting to indicate scoped collections. For example, for the case above where a
`dyno` belongs to an `app`, which belongs to an `org`, this API structure is
preferable:

```
/orgs/{org_id}
/orgs/{org_id}/apps
/apps/{app_id}
/apps/{app_id}/dynos
/dynos/{dyno_id}
```

### URLs for Resource Collections

> **Do** form the URL for a collection of resources from the resource type.

For example, a collection of resources of type `photos` will have the URL:

```
/photos
```

### URLs for Individual Resources

> **Do** treat collections of resources as sets keyed by resource ID.

The URL for an individual resource can be formed by appending the resource’s ID
to the collection URL.

For example, a photo with an ID of `"1"` will have the URL:

```
/photos/1
```

### Relationship URLs and Related Resource URLs

> **Do** follow the relationship and related resource URL guidelines below, if
> our API supports relationship URLs and/or related resource URLs.

> **Consider** forming a related resource URL by appending the name of the
> relationship to the resource’s URL.

> **Consider** forming a relationship URL by appending `/relationships/` and the
> name of the relationship to the resource’s URL.

As described in the [JSON API Specification](#json-api-specification), there are
two URLs that can be exposed for each relationship:

* "Related Resource URL" – a URL for the related resource(s), which is
  identified with the related key within a relationship’s links object. When
  fetched, it returns the related resource object(s) as the response’s primary
  data.
* "Relationship URL" – a URL for the relationship itself, which is identified
  with the self key in a relationship’s links object. This URL allows the client
  to directly manipulate the relationship. For example, it would allow a client
  to remove an author from a post without deleting the people resource itself.

For example, the URL for a photo’s comments will be:

```
/photos/1/comments
```

And the URL for a photo’s photographer will be:

```
/photos/1/photographer
```

A photo’s `comments` relationship will have the URL:

```
/photos/1/relationships/comments
```

And a photo’s `photographer` relationship will have the URL:

```
/photos/1/relationships/photographer
```

### Canonical Identifier

> **Do** expose a URL containing a stable, unique identifier for any and all
> resources which can be moved or renamed.

> **Consider** exposing a URL containing a stable, unique identifier for all
> resources.

> **Consider** using a GUID as the stable identifier for resources, where
> appropriate (e.g. when backed by a non-relational database).

It MAY be necessary to interact with the service to obtain a stable URL from
the friendly name for the resource, as in the case of the `/my` shortcut used
by some applications.

An example of a URL containing a friendly name is:

```
https://api.example.com/people/jdoe@contoso.com
```

And an example of a URL containing a canonical identifier is:

```
https://api.example.com/people/10f7a66b-083d-401f-93e7-dffc172c6fea
```

### URL Controllers and Verbs

> **Avoid** using CRUD HTTP methods or verbs (e.g. GET, DELETE, etc.) in URL
> paths.

URIs SHOULD NOT be used to indicate that a CRUD function is performed. URIs
SHOULD be used to uniquely identify resources, and they SHOULD be named as
described in the sections below. HTTP methods and verbs MUST be used to indicate
which CRUD function is performed.

For example, this API interaction design is valid:

```http
DELETE /users/1234
```

While the following anti-patterns exemplify what not to do:

```http
GET /deleteUser?id=1234
GET /deleteUser/1234
DELETE /deleteUser/1234
POST /users/1234/delete
```

See the [Methods section](#methods) below for more details.

### URL Length

The HTTP 1.1 message format, defined in
[IETF RFC 7230 Section 3.1.1][rfc-7230-3-1-1], defines no length limit on the
Request Line, which includes the target URL.

> **Do** respond with a  414 (URI Too Long) status code to any request with a
> target URI longer than we wish to parse.

> **Consider** making accommodations for the clients we wish to support when
> implementing a service which generates URLs longer than 2,083 characters.

_**Note:** Some technology stacks have hard and adjustable URL limits, so keep
this in mind as you design services with longer than usual URLs._

* http://stackoverflow.com/a/417184
* https://blogs.msdn.microsoft.com/ieinternals/2014/08/13/url-length-limits

## Methods

> **Do** use the proper HTTP methods for API operations whenever possible.

> **Do** respect operational idempotency (see the table below).

Below is a list of methods RESTful HTTP APIs / services SHOULD support. Not all
resources will support all methods, but all resources using one or more of the
methods below MUST conform to their usage.

Method  | Description                                                                                                                  | Is Idempotent
------- | ---------------------------------------------------------------------------------------------------------------------------- | -------------
GET     | Return the current value of an object.                                                                                       | True
PUT     | Replace an object, or create a named object, when applicable.                                                                | True
DELETE  | Delete an object.                                                                                                            | True
POST    | Create a new object based on the data provided, or submit a command.                                                         | False
HEAD    | Return metadata of an object for a GET response. Resources which support the GET method MAY support the HEAD method as well. | True
PATCH   | Apply a partial update to an object.                                                                                         | False
OPTIONS | Get information about a request; see below for details.                                                                      | True

### DELETE

> **Consider** supporting deletion of multiple resources at once via a
> GET-style filtering of resource IDs (e.g. `/users?id=[20,30,40,60,90]`).

> **Consider** supporting deletion of multiple resources at once via a
> comma-separated list of resource IDs (e.g. `/users/20,30,40,60,90`).

Deleting resources is defined as part of the
[JSON API Specification](#json-api-specification). See the link below for full
DELETE implementation requirements and specification details.

<http://jsonapi.org/format/#crud-deleting>

### GET

Fetching resources is defined as part of the
[JSON API Specification](#json-api-specification). See the link below for full
GET implementation requirements and specification details.

<http://jsonapi.org/format/#fetching-resources>

### OPTIONS

> **Do** respond to OPTIONS requests with an Allow header denoting the valid
> methods for the resource.

> **Consider** including a Link response header to point to documentation for
> the requested resource.

OPTIONS allows a client to retrieve information about a resource. See
[IETF RFC 5988][rfc-5988] for more information on Link headers.

For example, in an OPTIONS respond we MAY include:

```http
Link: <{help}>; rel="help"
```

Where `{help}` is the URL to a documentation resource.

For more examples OPTIONS use cases, see
[preflighting CORS cross-domain calls][preflight-requests].

### PATCH

PATCH has been standardized by IETF as the method used for incrementally
updating an existing object (see [IETF RFC 5789][rfc-5789]).

> **Do** support creation of resources using PATCH, if our service DOES support
> UPSERT semantics.

> **Do** respond with an HTTP "409 Conflict" error code to a PATCH request
> against a resource that does not exist, if our service DOES NOT support
> UPSERT semantics.

> **Consider** supporting UPSERT semantics in services which allow callers to
> specify key values on create.

> **Consider** supporting updates to multiple resources at once via a
> GET-style filtering of resource IDs (e.g. `/users?id=[20,30,40,60,90]`).

> **Consider** supporting updates to multiple resources at once via a
> comma-separated list of resource IDs (e.g. `/users/20,30,40,60,90`).

> **Avoid** treating a PATCH request as an insert if it contains an If-Match
> header.

> **Avoid** treating a PATCH request as an update if it contains an
> If-None-Match header with a value of "\*".

Under UPSERT semantics, a PATCH call to a nonexistent resource is handled by
the server as a "create," and a PATCH call to an existing resource is handled
as an "update." To ensure that an update request is not treated as a create or
vice-versa, the client MAY specify precondition HTTP headers in the request.

Updating resources is defined as part of the
[JSON API Specification](#json-api-specification). See the link below for full
PATCH implementation requirements and specification details.

<http://jsonapi.org/format/#crud-updating>

### POST

> **Do** support the Location response header to specify the location of any
> created resource that was not explicitly named, via the Location header.

> **Consider** returning full metadata for the created item in the response.

As an example, imagine a service that allows creation of hosted servers, which
will be named by the service:

```http
POST https://api.example.com/account1/assets
```

The response should resemble:

```http
201 Created
Location: https://api.example.com/account1/assets/asset321
```

Where `asset321` is the service-allocated Asset name.

Creating resources is defined as part of the
[JSON API Specification](#json-api-specification). See the link below for full
POST implementation requirements and specification details.

<http://jsonapi.org/format/#crud-creating>

### PUT

> **Do** use replacement semantics (i.e after the PUT, the resource's
> properties MUST match what was provided in the request, including deleting
> any extant properties which are not provided), for any service which supports
> PUT requests.

> **Consider** supporting PUT requests to update (via replacement semantics)
> existing resources.

Because PUT is defined as a complete replacement of the content, it is
dangerous for clients to use PUT to modify data. Clients that do not understand
(and hence ignore) properties on a resource are not likely to provide them on a
PUT when trying to update a resource, hence such properties MAY be
inadvertently removed.

See the [PATCH](#patch) section above for more details.

## Request Headers

> **Do** support request headers consistently across all endpoints and
> resources.

> **Consider** supporting each of the requests headers defined in the following
>  table.

The following table of request headers below SHOULD be supported by RESTful
HTTP APIs and services. Supporting all of these headers is not mandatory, but
if supported they MUST be supported consistently.

All header values MUST follow the syntax rules set forth in the specifications
below, where each header field is defined. Many HTTP headers are defined in
[IETF RFC 7231][rfc-7231], however a complete list of approved headers can be
found in the [IANA Header Registry][header-registry].

Header                            | Type                                  | Description
--------------------------------- | ------------------------------------- | -----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
Authorization                     | String                                | Authorization header for the request.
Date                              | Date                                  | Timestamp of the request, based on the client's clock, in [RFC 5322][rfc-5322-3-3] date and time format. The server SHOULD NOT make any assumptions about the accuracy of the client's clock. This header MAY be included in the request, but MUST be in this format when supplied. Greenwich Mean Time (GMT) MUST be used as the time zone reference for this header when it is provided. For example: `Wed, 24 Aug 2016 18:41:30 GMT`. Note that GMT is exactly equal to UTC (Coordinated Universal Time) for this purpose.
Accept                            | Content type                          | The requested content type for the response such as: <ul><li>application/json; version=1.0</li><li>application/vnd.api+json; version=1.0</li><li>application/xml; version=1.0</li><li>text/javascript; version=1.0</li></ul> Per the HTTP guidelines, this is just a hint and responses MAY have a different content type, such as a blob fetch where a successful response will just be the blob stream as the payload. Must also include requested API version.
Accept-Encoding                   | Gzip, deflate                         | REST endpoints SHOULD support GZIP and DEFLATE encoding, when applicable. For very large resources, services MAY ignore and return uncompressed data.
Accept-Language                   | "en", "es", etc.                     | Specifies the preferred language for the response. Services are not required to support this, but if a service supports localization it MUST do so through the Accept-Language header.
Accept-Charset                    | Charset type like "UTF-8"             | Default is UTF-8, but services SHOULD be able to handle ISO-8859-1.
Content-Type                      | Content type                          | Mime type of request body (PUT/POST/PATCH).
Prefer                            | return=minimal, return=representation | If the return=minimal preference is specified, services SHOULD return an empty body in response to a successful insert or update. If return=representation is specified, services SHOULD return the created or updated resource in the response. Services SHOULD support this header if they have scenarios where clients would sometimes benefit from responses, but sometimes the response would impose too much of a hit on bandwidth.
If-Match, If-None-Match, If-Range | String                                | Services that support updates to resources using optimistic concurrency control MUST support the If-Match header to do so. Services MAY also use other headers related to ETags as long as they follow the HTTP specification.

## Response Headers

> **Do** return all of response headers defined in the following table, except
> where noted in the "required" column.

Response Header    | Required                                                                   | Description
------------------ | -------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
Date               | All responses.                                                             | Timestamp the response was processed, based on the server's clock, in [RFC 5322][rfc-5322-3-3] date and time format. This header MUST be included in the response. Greenwich Mean Time (GMT) MUST be used as the time zone reference for this header. For example: `Wed, 24 Aug 2016 18:41:30 GMT`. Note that GMT is exactly equal to UTC (Coordinated Universal Time) for this purpose.
Content-Type       | All responses.                                                             | Content type.
Content-Encoding   | All responses.                                                             | GZIP or DEFLATE, as appropriate.
ETag               | When requested resource has an entity tag.                                 | ETag response-header field provides the current value of the entity tag for the requested variant. Used with If-Match, If-None-Match and If-Range to implement optimistic concurrency control.
Preference-Applied | When specified in request.                                                 | Whether a preference indicated in the Prefer request header was applied.
X-API-Version      | All responses.                                                             | Return the version of the API responding to the request.
X-API-Warn         | When deprecated API version is requested, or endpoint has been deprecated. | Header value should include: `WARNING! You are using a deprecated version of this API. For information on upgrading see <{link}>.`, where `{link}` is a URL to a relevant API documentation resource.

## Custom Headers

> **Avoid** requiring custom headers for the basic operation of an API.

Some of the guidelines in this document prescribe the use of domain-specific
HTTP headers. In addition, some services MAY expose extra functionality via
custom HTTP headers. The following guidelines help maintain consistency across
usage of custom headers.

Headers that are not standard HTTP headers MUST have one of two formats:

1. A generic format for headers IANA-registered as "provisional"
(see [IETF RFC 3864][rfc-3864]).
2. A scoped format for headers too usage-specific for registration.

@TODO: Clarify what is required above / provide an example of a custom header.

## Response Formatting

For organizations to have a successful platform, they must serve data
consistently and in formats developers are accustomed to using, allowing
developers to handle responses with common code.

### Golden Rules

If we are ever in doubt on an API formatting decision, may these Golden Rules
help guide us to the correct decision.

1. **Flat is better than nested.**
2. **Simple is better than complex.**
3. **Strings are better than numbers.**
4. **Consistency is better than customization.**

### Basics

> **Do** provide JSON as the default API response encoding.

> **Do** camelCase JSON property names.

> **Do** default to "application/json" response formatting if no Accept header
> is provided.

> **Consider** supporting alternative response formats requested by the client
> using the Accept header.

> **Consider** returning a 406 Not Acceptable HTTP error code when rejecting a
> client request due an incompatible or malformed Accept request header.

In HTTP, response format SHOULD be requested by the client using the Accept
header. This is a hint, and the server MAY ignore it if it chooses to, even if
this isn't typical of well-behaved servers. Clients MAY send multiple Accept
headers and the service MAY choose one of them.

The default response format (no Accept header provided) SHOULD be
`application/json`, and all services MUST support `application/json`.

Accept Header            | Response Type                                                     | Notes
------------------------ | ----------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------
application/json         | Payload SHOULD be returned in JSON format.                        | Also accept text/javascript for JSONP cases.
application/vnd.api+json | Payload SHOULD be returned as [JSON API format][json-api-format]. | MAY be used interchangably with `application/json` for services which support JSON API as a first-class format.

Request example.

```http
GET https://api.example.com/products/storage
Accept: application/json
```

See the [JSON API Specification section](#json-api-specification) below for full
response formatting details.

### Date Formatting

@TODO: This section is a stub. Consider expanding.

> **Do** use the `Iso8601Literal` date format, unless there are very compelling
> reasons to do otherwise.

[This W3C NOTE][w3c-date-format] provides an overview of the recommended
formats.

### JSON API Specification

> **Do** adhere to the JSON API Specification formatting guidelines, detailed
> below, as closely as possible.

> **Do** respond to requests with "application/json" Accept headers with JSON
> API formatted data and a "application/vnd.json-api" Content-Type header, if
> no other non-vendored JSON format is supported by the API.

> **Consider** supporting alternative JSON response formats, as necessitated by
> application or business requirements.

JSON API is a specification for how a client should request that resources be
fetched or modified, and how a server should respond to those requests.

Familiarity with the JSON API specification is **strongly encouraged**.

We SHOULD model our API response structures on JSON API as closely as possible.
The full specification along with examples and popular libraries can be
referenced at the link below.

<http://jsonapi.org>

### Booleans, NULLs and Stringification

> **Do** format all binary field types as boolean.

> **Do** format all numbers as strings in the response body, including numeric
> resource IDs and primary keys.

> **Do** format booleans as booleans (non-strings), and nulls as nulls
> (non-strings).

> **Do** return NULL for fields with no value, when a value has been deleted,
> and when a value never existed.

> **Do** return NULL for non-existing boolean, number and string field values.

> **Avoid** representing string fields without value as `""`. Return a NULL
> value instead (they mean two different things).

> **Avoid** returning NULL for empty arrays as sets. Instead, return a literal
> empty array (`[]`) and/or set (`{}`).

> **Avoid** typecasting field values to numbers. This is almost never a good
> idea, as it often leads to double-casting, hidden inefficiencies, testing
> complexity and client difficulties.

These guidelines exist to avoid confusion, minimize complexity and make front
end developers’ work a lot easier.

Remember, a field type shall be boolean if its value is binary.

Don’t use `0` and `1`, don’t use `"0"` and `"1"`, and don’t try to come up with
a better solution. Use what we already have: `true` and `false` (type cast here
need be). It’s universal and objective.

Example:

```http
GET /customers/16784

{
  "type": "customers",
  "id": "16784",
  "attributes": {
    "name": "Joe Smith",
    "age": null,
    "relatives": [],
    "address": null
  }
}
```

> **Note:** "id" is represented as a string.

Much like date/time localization, type casting and type transformation should
only happen once, and as close to the client as possible. Trust the client to
know how it wants its data formatted, and save ourselves unnecessary work by
returning field values as glorious, JavaScript-native "Strings" wherever
feasible.

## Filtering

> **Do** reserve the `filter` query parameter to be used as the basis for any
> filtering strategy.

> **Consider** supporting filtering of a resource collection based upon
> associations by allowing query parameters that combine `filter` with the
> association name.

For example, the following is a request for all comments associated with a
particular post:

```http
GET /comments?filter[post]=1 HTTP/1.1
```

Multiple filter values can be combined in a comma-separated list. For example:

```http
GET /comments?filter[post]=1,2 HTTP/1.1
```

Furthermore, multiple filters can be applied to a single request:

```http
GET /comments?filter[post]=1,2&filter[author]=12 HTTP/1.1
```

See the [JSON API Specification section](#json-api-specification) above for
full filtering implementation requirements and specification details.

<http://jsonapi.org/format/#fetching-filtering>

## Pagination

> **Consider** limiting the number of resources returned in a response to a
> subset (“page”) of the whole set available.

> **Consider** providing links to traverse a paginated data set (“pagination
> links”).

Here's an illustration of a page-based implementation of pagination links.

Example Request:

```http
GET /articles?page[number]=3&page[size]=1 HTTP/1.1
```

Example Response:

```json
HTTP/1.1 200 OK
Content-Type: application/vnd.api+json

{
  "meta": {
    "total-pages": 13
  },
  "data": [
    {
      "type": "articles",
      "id": "3",
      "attributes": {
        "title": "JSON API paints my bike shed!",
        "body": "The shortest article. Ever.",
        "created": "2015-05-22T14:56:29.000Z",
        "updated": "2015-05-22T14:56:28.000Z"
      }
    }
  ],
  "links": {
    "self": "https://example.com/articles?page[number]=3&page[size]=1",
    "first": "https://example.com/articles?page[number]=1&page[size]=1",
    "prev": "https://example.com/articles?page[number]=2&page[size]=1",
    "next": "https://example.com/articles?page[number]=4&page[size]=1",
    "last": "https://example.com/articles?page[number]=13&page[size]=1"
  }
}
```

Pagination is defined as part of the
[JSON API Specification](#json-api-specification). See the link below for full
pagination implementation requirements and specification details.

<http://jsonapi.org/format/#fetching-pagination>

## Sorting

> **Do** sort NULL values as "less than" non-NULL values, for services which
> support sorting.

> **Consider** supporting sorting the results of a collection query based on
> property values.

Sorting is defined as part of the
[JSON API Specification](#json-api-specification). See the link below for full
sorting implementation requirements and specification details.

<http://jsonapi.org/format/#fetching-sorting>

## Kong

@TODO.

## Monitoring

RESTful HTTP services MUST implement the `/health` and `/version` API endpoints
according to the guidelines defined below.

`/debug`, `/metrics` and `/status` endpoints are highly recommended.

### Health

> **Do** respond to requests to `/health` with a `200 OK` status code.

> **Do** respond with a `503 Service Unavailable` to indicate a severely
degraded or offline service.

> **Avoid** returning anything in the response body, or otherwise
over-complicating the `/health` endpoint.

`/health` endpoints are designed specifically for load balancers and service
discovery solutions. As such, they should be exceedingly simple, indicating the
health of an API via HTTP response codes.

A load balancer should know whether to keep a service in the pool or eject it
after sending a HEAD request to the service health endpoint.

Example Request:

```http
HEAD /health HTTP/1.1
```

Example Response:

```json
HTTP/1.1 200 OK
Date: Sat, 10 Sep 2012 20:50:55 GMT
X-API-Version: 1.0
```

### Debug

> **Do** disable `/debug` endpoint by default.

> **Do** require explicit `DEBUG=true` configuration or environment variable at
application start-up to enable `/debug` endpoint.

> **Avoid** exposing `/debug` endpoint in non-development environments.

`/debug` endpoints are designed specifically internal use, to expose sensitive
configuration, environment and logging information to aid in development. These
endpoints are NOT for public consumption, as such should be disabled by default,
requiring an explicit non-default configuration setting or environment variable
to enable at start-up.

Useful `/debug` endpoint responses might include.

* A copy of package.json.
* Date of last service restart.
* Last few debug-level log messages.

### Metrics

> **Do** expose application counters and statistics via the `/metrics` endpoint.

> **Avoid** calculating difficult-to-compute or resource-intensive metrics.

`/metrics` endpoints are designed specifically for dashboard tools and time
series database integrations. A metrics endpoint should produce live results,
and should be able to serve applications at 1 QPS without any impact on
application performance. As such, avoid exposing difficult-to-compute or
resource-intensive metrics, or if we must, average or cache them for as long as
necessary and name them accordingly (e.g. `response_time_avg_60_sec`).

Useful `/metrics` endpoint responses might include.

* Uptime in seconds.
* Average response time.
* Average QPS over last minute.
* Number of 500 errors.

### Status

> **Do** respond to requests to `/status` with a JSON object describing the
current service status.

> **Do** respond with a `500 Internal Server Error` to indicate a degraded or
offline dependency, when no more specific message is suitable.

> **Avoid** calculating difficult-to-compute or resource-intensive metrics.

`/status` endpoints are designed for both human consumption, and also for
dynamic load balancing. Outputs should be intuitive and easy-to-read, should
also be able to serve applications at 1 QPS without any impact on application
performance. Avoid exposing difficult-to-compute or resource-intensive values,
or if we must, background-compute and cache them for as long as necessary to
make the performance hit negligible.

Example Request:

```http
GET /status HTTP/1.1
```

Example Response:

```json
HTTP/1.1 500 Internal Server Error
Date: Sat, 10 Sep 2012 20:50:55 GMT
X-API-Version: 1.0

{
  "status": "degraded",
  "build": "2",
  "connections": {
    "database": "ok",
    "google": "ok",
    "files": "ok",
    "permissions": "down"
  },
  "restarted": "2016-11-10T10:30:00Z",
  "updated": "2016-10-30T13:15:30Z"
}
```

### Version

> **Do** respond to requests to `/version` with a JSON object describing running
version of the service.

> **Do** load application runtime version into memory at start-up, and then
serve the version value from memory.

> **Avoid** sourcing version information from files on disk, or other mutable
sources external to the application runtime.

Loads version information AT STARTUP, then serves from memory. Independent of
changes to files on disk.

`/version` endpoints are designed for both human consumption and dynamic service
discovery. Responses should be exceedingly simple, indicating the version of an
API and nothing else.

It is very important to ensure the application returns the _running_ version of
the application, and not the _installed_ version. Always load version
information into memory at application start-up, rather than sourcing from any
external or mutable location.

Example Request:

```http
GET /version HTTP/1.1
```

Example Response:

```json
HTTP/1.1 200 OK
Date: Sat, 10 Sep 2012 20:50:55 GMT
X-API-Version: 1.0

{
  "version": "1.0"
}
```

## Schema Documentation

Human-readable, machine-ingestible.

@TODO. API Blueprint, Swagger, etc.

## Asynchronous Processing and Long-Running Operations

> **Consider** returning a status `202 Accepted` with a link in the
> `Content-Location` header asynchronously for long-running operations.

Imagine a situation when we need to create a resource and the operation takes
long time to complete.

```http
POST /photos HTTP/1.1
```

The request SHOULD immediately return a status `202 Accepted` with a link in the
`Content-Location` header.

```json
HTTP/1.1 202 Accepted
Content-Type: application/vnd.api+json
Content-Location: https://example.com/photos/queue-jobs/5234

{
  "data": {
    "type": "queue-jobs",
    "id": "5234",
    "attributes": {
      "status": "Pending request, waiting other process"
    },
    "links": {
      "self": "/photos/queue-jobs/5234"
    }
  }
}
```

To check the status of the job process, a client can send a request to the
location provided in the `Content-Location` header.

```http
GET /photos/queue-jobs/5234 HTTP/1.1
Accept: application/vnd.api+json
```

When job process is done, the request SHOULD return a status `303 See other`
with a link in Location header.

```http
HTTP/1.1 303 See other
Content-Type: application/vnd.api+json
Location: https://example.com/photos/4577
```

## Push Notifications

@TODO.

## Testing.

@TODO. Unit tests should run independently, without requiring an internet
connection, database, or any other running service. Integration tests should
encapsulate tests between dependencies. Acceptance tests should verify
end-to-end functionality.

All three should be invokable separately.

## Versioning

> **Do** require the client to specify the version as part of the media type in
> the Accept request header.

Versioning and the transition between versions can be one of the more
challenging aspects of designing and operating an API. As such, it is best to
build in some mechanisms to mitigate this pain point from the start.

To prevent surprises and breaking changes to API consumers, it is best to
require a version be specified with all requests. Default versions should be
avoided as they are very difficult to change in the future (by design, any
change to the default version would be a breaking change).

`Accept` header versioning requires a version to be included as a media type
parameter that supplements the main media type in every client request.

Here's an example HTTP request using the accept header versioning style.

```http
GET /bookings/ HTTP/1.1
Host: example.com
Accept: application/json; version=1.0
```

In the example request above `request.version` attribute would return the string
`"1.0"`.

Here's another example HTTP request requesting the JSON API vendor media type.

```http
GET /bookings/ HTTP/1.1
Host: example.com
Accept: application/vnd.api+json; version=1.0
```

Versioning based on `Accept` headers with a [vendor media type][drf-1] is
[generally considered][nobody-understands] as
[best practice][require-versioning].

### Breaking Change

Changes to the contract of an API are considered a breaking change. Changes that
impact the backwards compatibility of an API are a breaking change. Typically,
adding of endpoints, resources and fields is not considered breaking, while the
modification or removal of the same is considered breaking.

Clear examples of breaking changes:
* Removing or renaming APIs or API parameters.
* Changes in behavior for an existing API.
* Changes in Error Codes and Fault Contracts.
* Anything that would violate the
  [Principle of Least Astonishment][least-astonishment].

### Deprecation

> **Do** always return the API version responding to the request in an
> `X-API-Version` response header.

> **Do** include an `X-API-Warn` response header when a deprecated API version
> is requested in the `Accept` request header, or endpoint has been deprecated.

> **Do** include a warning in the `meta` member of the JSON response if a
> request is made with deprecated fields or query parameters.

> **Do** indicate in the API documentation when deprecated API versions will
> reach end-of-life.

In a deprecated requested API version or endpoint scenario, return an
`X-API-Warn` response header with a value which looks something like this:

> _WARNING! You are using a deprecated version of this API. For information on
> upgrading see <{link}>._

In a deprecated field or query parameter scenario, return a warning in the
`meta` member of the JSON response which looks something like this:

```json
{
  "meta": {
    "warning": "You are using a deprecated version of this API. For information on upgrading see <{link}>."
  },
  "data": {
    // ...
  }
}
```

Where `{link}` is a URL to a relevant API documentation resource.

The idea is this message will appear in server logs, hopefully prompting API
consumers to upgrade.

### How and When to Version

> **Do** increment an API's version number in response to any breaking change.

> **Do** indicate the current support status of each API version, past and
> present, in the API's documentation.

> **Do** support version numbers in the format `{majorVersion}.{minorVersion}`.

> **Consider** incrementing an API's version number for non-breaking changes.

Use a new major version number to signal that support for existing clients will
be deprecated in the future. When introducing a new major version, services MUST
provide a clear upgrade path for existing clients and develop a plan for
deprecation that is consistent with their business group's policies. Services
SHOULD use a new minor version number for all other changes.

Online documentation of versioned services MUST indicate the current support
status of each previous API version.

#### Group Versioning

> **Consider** supporting group versioning across multiple microservices to
> provide multi-API consumers with a unified versioning scheme.

Group versions allow for logical grouping of API endpoints under a common
versioning moniker. This allows developers to look up a single version number
and use it across multiple endpoints. Group version numbers are well known, and
services SHOULD reject any unrecognized values.

Internally, each services will take a Group Version and map to its appropriate
Major.Minor version.

The Group Version format is defined as YYYY-MM-DD, for example 2012-12-01 for
December 1, 2012. This Date versioning format applies only to Group Versions and
SHOULD NOT be used as an alternative to Major.Minor versioning.

Examples:

| Group      | API Major.Minor   |
| ---------- | ----------------- |
| 2012-12-01 | Assets 1.0        |
|            | Files 1.1         |
|            | Projects 1.2      |
| 2013-03-21 | Assets 1.0        |
|            | Files 2.0         |
|            | People 3.0        |
|            | Players 3.1       |
|            | Projects 3.2      |
|            | Users 3.3         |

Clients can specify either the group version or the endpoint-specific
Major.Minor version.

For example:

```http
GET /assets/ HTTP/1.1
Host: api.example.com
Accept: application/vnd.api+json; version=1.0
```

Or equivalently, when Group Verions are supported:

```http
GET /assets/ HTTP/1.1
Host: api.example.com
Accept: application/vnd.api+json; version=2012-12-01
```

All supported Group Versions, along with their deprecation statuses and
end-of-life dates, should be published and kept up-to-date in the public API
documentation.

## Security

> **Do** enforce HTTPS (TLS-encrypted) across all endpoints, resources and
> services.

> **Do** enforce and require HTTPS for all callback URLs, push notification
> endpoints and web hooks.

### CORS

> **Do** support CORS (Cross Origin Resource Sharing) headers for all
> public-facing APIs.

> **Consider** supporting a CORS allowed origin of "\*", and enforcing
> authorization through valid OAuth tokens.

> **Avoid** combining user credentials with origin validation.

There MAY be exceptions for special cases.

### PII Parameters

> **Do** obfuscate any PII parameters passed in from clients before logging
> the request.

> **Consider** accepting PII parameters transmitted from the client to the
> server in the form of HTTP headers.

> **Avoid** accepting PII parameters transmitted from the client to the server
> in the URL (as part of path or query string).

Consistent with best practice privacy policies, clients SHOULD NOT transmit
personally identifiable information (PII) parameters in the URL (as part of path
or query string) because this information can be inadvertently exposed via
client, network, and server logs and other mechanisms. Similarly, a server MUST
obfuscate any PII parameters passed in from clients before logging the request.

A service SHOULD accept PII parameters transmitted as headers.

Services which accept PII parameters as headers MUST be compliant with privacy
policy standards dictated by common sense and best practices. Services accepting
PII parameters must adhere to special precautions to ensure that logs and other
service data collection are properly handled.

If we think our API or service may be handling PII parameters, we MUST
consult with and seek approval from engineering leadership before releasing
our service to customers.

### Secure Connections

> **Do** require secure connections with TLS to access our APIs, without
> exception.

> **Avoid** responding to any non-TLS requests for HTTP or over port 80 to avoid
> any insecure data exchange.

It’s not worth trying to figure out or explain when it is OK to use TLS and when
it’s not. Just require TLS for everything.

Reject any non-TLS requests by not responding to requests for HTTP or over port
80 to avoid any insecure data exchange. In environments where this is not
possible, respond with 403 Forbidden.

Redirects are discouraged since they permit sloppy client behavior, without
providing any clear gain. Clients relying on redirects double up on server
traffic, and render TLS impotent since sensitive data will already have been
exposed during the first unencrypted call.

## Taxonomy

RESTful HTTP services MUST comply with the taxonomy guidelines defined below.

### Errors

Errors, or more specifically Service Errors, occur when a client makes an
invalid or incorrect request to a service, or passes invalid or incorrect data
to a service, and the service rejects the request. Examples include invalid
authentication credentials, incorrect parameters, unknown version IDs, etc.

> **Do** return "4xx" HTTP error codes when rejecting a client request due to
> one or more Service Errors.

> **Consider** processing all attributes and then returning multiple validation
> problems in a single response.

> **Consider** implementing fast-failing request validation—which protects a
> service from especially resource-intensive requests, for example—as Service
> Errors.

> **Avoid** returning Service Errors to communicate overall API availability
> (e.g. Too Many Requests). Guidelines for communicating service availability
> are defined in a later section (see [Faults](#faults)).

See the [JSON API Specification section](#json-api-specification) above for
full error requirements and specification details.

* <http://jsonapi.org/format/#errors>
* <http://jsonapi.org/examples/#error-objects>

### Faults

Faults, or more specifically Service Faults, occur when a service fails to
correctly respond to a valid client request.

> **Do** return "5xx" HTTP error codes when rejecting a client request due to
> one or more Service Faults.

> **Avoid** returning Service Faults to communicate invalid or incorrect
> request formatting or data from a client. Guidelines for communicating >
> validation failures are defined in a previous section (see [Errors](#errors)).

> **Avoid** returning Service Faults to communicate rate limiting or quota
> failures.

## Metrics

> **Consider** calculating and emitting time series averages for each of the
> following metrics.

### Availability

Availability is typically measured in terms of uptime (service is reachable),
and the ratio of successful operations to faults. This is the single most
important metrics for services to track; measuring and emitting these metrics
within and as part of the service runtime is highly recommended.

### Latency

Latency is defined as how long a particular API call takes to complete, measured
as closely to the client as possible. This metric applies to both synchronous
and asynchronous APIs in the same way. For long running calls, the latency is
measured on the initial request and measures how long that call (not the overall
operation) takes to complete.

### Time to Complete

Services that expose long operations MUST track "Time to Complete" metrics
around those operations.

### Long Running API Faults

For a Long Running API call, it's possible for both the initial request to begin
the operation and the request to retrieve the results to technically work (each
passing back a 200), but for the underlying operation to have failed. Long
Running faults MUST roll up as Faults into the overall Availability metrics.

## Client Guidelines

To ensure the best possible experience for clients interfacing with a RESTful
HTTP service, clients SHOULD adhere to the following best practices.

### Ignore Rule

> **Do** safely ignore unknown and unexpected data in API responses.

> **Do** safely ignore additional fields and object attributes in API responses.

Some services MAY add fields to responses without changing versions numbers.
Services that do so MUST make this clear in their documentation and clients
MUST ignore unknown fields.

### Silent Failures

> **Avoid** tightly coupling client actions to optional service functionality.

Clients requesting OPTIONAL service functionality (such as optional headers)
MUST be resilient to the server ignoring or not supporting that particular
functionality.

### Variable Ordering

> **Do** rely only on ordering behavior explicitly defined in the service
> contract and identified by the service.

> **Consider** explicitly specifying the ordering of order-sensitive arrays
> and elements as part of the service contract.

> **Avoid** relying on the order in which data appears in API JSON responses,
> unless order is explicitly defined as part of the service contract.

Clients SHOULD be resilient to the reordering of fields within a JSON object.
When supported by the service, clients MAY request that data be returned in a
specific order. For example, services MAY support the use of the $orderBy query
string parameter to specify the order of elements within a JSON array.

## Miscellaneous

@TODO.

## Reference APIs and Libraries

@TODO. Link to examples of well-designed APIs, exemplifying the best practices
and standards outlined in this document.

### APIs

* [RestPack](http://restpack-serializer-sample.herokuapp.com)

### Libraries

#### Node

* [LoopBack](https://loopback.io)
* [swagger-tools](https://www.npmjs.com/package/swagger-tools)

#### Python

* [Django REST Framework](http://www.django-rest-framework.org)

## FAQ

## See Also

Understanding the philosophy behind the REST Architectural Style is recommended
for developing HTTP-based services. If you are new to RESTful design, here are
some good introductory resources:

[IETF RFC 7231][rfc-7231] — Defines the specification for HTTP/1.1 semantics,
and is considered the authoritative resource.

[REST Dissertation][rest-disseration] — The chapter on REST in Roy Fielding's
dissertation on Network Architecture, "Architectural Styles and the Design of
Network-based Software Architectures"

[REST in Practice][rest-in-practice] — Book on the fundamentals of REST.

[REST on Wikipedia][wikipedia-rest] — Overview of common definitions and core
ideas behind REST.

## References

* <http://jsonapi.org/recommendations>
* <http://shop.oreilly.com/product/0636920021575.do>
* <https://geemus.gitbooks.io/http-api-design/content/en/>
* <https://github.com/Microsoft/api-guidelines>

# Drawbacks

* Conventions and standards implicitly involve some overhead. Standards must be
maintained and conventions modernized from time to time, and when they do change
a multiplicity of projects are often impacted. Ideally, conventions and
standards do not change very often.

# Alternatives

What other designs have been considered? What is the impact of not doing this?

## Impact of Not Standardizing

* Integration costs quickly become unmanageable.

  As microservices proliferate, standards become the contracts around which
  automation and economies of scale are built and implemented. Writing a generic
  API schema consumer, or a new automation role, should require critical
  thinking and standards review only once, and then should be rinse-and-repeat
  implementable everywhere.

  This is what conventions and standards afford us.

  Without a set of adopted standards, implementing each API consumer requires
  critical thinking and schema review, each automation role requires a fresh
  approach and service-specific tweaks, and the company cannot reap the
  economies of scale implicit in a unified approach. Bridging and integrating
  APIs and services requires custom interfaces on both ends, often resulting in
  a full mesh each-api-and-service-for-itself integration model—an exercise
  which costs time, money, and provides no tangible business value.

# Unresolved Questions

Coming soon...

[drf-1]: http://www.django-rest-framework.org/api-guide/versioning/#using-accept-headers-with-vendor-media-types
[header-registry]: http://www.iana.org/assignments/message-headers/message-headers.xhtml
[json-api-format]: http://jsonapi.org/format/
[least-astonishment]: https://en.wikipedia.org/wiki/Principle_of_least_astonishment
[nobody-understands]: http://blog.steveklabnik.com/posts/2011-07-03-nobody-understands-rest-or-http#i_want_my_api_to_be_versioned
[preflight-requests]: http://www.w3.org/TR/cors/#resource-preflight-requests
[require-versioning]: https://github.com/interagent/http-api-design/blob/master/en/foundations/require-versioning-in-the-accepts-header.md
[rest-disseration]: https://www.ics.uci.edu/~fielding/pubs/dissertation/rest_arch_style.htm
[rest-in-practice]: http://www.amazon.com/REST-Practice-Hypermedia-Systems-Architecture/dp/0596805829
[rfc-2119]: https://tools.ietf.org/html/rfc2119
[rfc-5322-3-3]: https://tools.ietf.org/html/rfc5322#section-3.3
[rfc-5789]: https://tools.ietf.org/html/rfc5789
[rfc-5988]: https://tools.ietf.org/html/rfc5988
[rfc-7230-3-1-1]: https://tools.ietf.org/html/rfc7230#section-3.1.1
[rfc-7231]: https://tools.ietf.org/html/rfc7231
[w3c-date-format]: https://www.w3.org/TR/NOTE-datetime
[wikipedia-rest]: http://en.wikipedia.org/wiki/Representational_state_transfer
