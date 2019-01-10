# Fourth API Guidelines

## 0. TL;DR

These guidelines, as a design principle, encourage Fourth developers to create APIs that provide a consistent user experience for the consumers of our APIs.

## 1. API Strategy

We have set up the following overarching strategy:

* Authentication happens at the API Gateway level.  All calls reaching the API will already be authenticated.
* OAuth is our Authentication method of choice, although we will continue to support basic authentication
* For APIs that are authenticated by OAuth, a X-Fourth-JWT header will be appended containing the validated, decoded JWT bearer token
* Authorization happens at the API, the API Gateway will append X-Fourth-Org header containing the Customer Account ID and the X-Fourth-UserID header containing the User ID
* All HTTP API's must provide Swagger (OpenAPI) documentation
* Leverage the benefits of HTTP such as statelessness, caching, gzipping, consistent verb usage etc. See this [HTTP Overview](https://developer.mozilla.org/en-US/docs/Web/HTTP/Overview)
* Two types of API have been identified
  * CRUD style APIs that expose resources
  * Batch style APIs that are expose denormalised, flattened structures useful for integrations to large systems such as SAP

## 2. Table of contents

<!-- TOC -->

- [Fourth API Guidelines](#fourth-api-guidelines)
    - [0. TL;DR](#0-tldr)
    - [1. API Strategy](#1-api-strategy)
    - [2. Table of contents](#2-table-of-contents)
    - [3. Introduction](#3-introduction)
        - [Design Principles](#design-principles)
            - [Build for the consumer](#build-for-the-consumer)
            - [Convention over configuration](#convention-over-configuration)
            - [Optimize close to the metal](#optimize-close-to-the-metal)
    - [4. Interpreting the guidelines](#4-interpreting-the-guidelines)
        - [4.1. Application of the guidelines](#41-application-of-the-guidelines)
        - [4.2. Guidelines for existing services and versioning of services](#42-guidelines-for-existing-services-and-versioning-of-services)
        - [4.3. Requirements language](#43-requirements-language)
    - [5. Archetypes](#5-archetypes)
    - [Docroot](#docroot)
        - [Resource](#resource)
        - [Collection](#collection)
        - [Controller](#controller)
    - [5. Structures](#5-structures)
        - [Dates and Times](#dates-and-times)
    - [6. Error Handling](#6-error-handling)
        - [6.1 Return an HTTP status code that closely matches the error condition](#61-return-an-http-status-code-that-closely-matches-the-error-condition)
        - [6.2 Return human-readable error messages](#62-return-human-readable-error-messages)
        - [6.3 Return machine-readable error codes](#63-return-machine-readable-error-codes)
        - [6.4 Return the Invalid parameter name](#64-return-the-invalid-parameter-name)
        - [6.5 Set Location](#65-set-location)
        - [6.6 Provide a Help URL](#66-provide-a-help-url)
            - [Example JSON Error Response:](#example-json-error-response)
    - [7. Swagger API Documentation](#7-swagger-api-documentation)
    - [8. Layer 7](#8-layer-7)
    - [9. Collections](#9-collections)
        - [Collection Query Syntax](#collection-query-syntax)
        - [Collection Response Structure](#collection-response-structure)
    - [10. URL design](#10-url-design)
        - [Nouns](#nouns)
        - [Verbs](#verbs)
        - [Plural resource nouns](#plural-resource-nouns)
        - [Nesting resources](#nesting-resources)
        - [Grammar](#grammar)
    - [11. HTTP Response Codes](#11-http-response-codes)

<!-- /TOC -->

## 3. Introduction

A goal of these guidelines is to ensure that Fourth APIs can be easily and consistently consumed by any client with basic HTTP support.

To provide the smoothest possible experience for developers, it's important to have these APIs follow consistent design guidelines, thus making using them easy and intuitive.
This document establishes the guidelines to be followed by Fourth API developers for developing such APIs consistently.

The benefits of consistency accrue in aggregate as well; consistency allows teams to leverage common code, patterns, documentation and design decisions.

These guidelines aim to achieve the following:

* Define consistent practices and patterns for all API endpoints across the Fourth estate.
* Adhere closely to accepted REST/HTTP best practices
* Make accessing Fourth Services via HTTP interfaces easy for all application developers.
* Allow service developers to leverage the prior work of other services to implement, test and document REST endpoints defined consistently.

### Design Principles

If we are ever in doubt on an API design or implementation choice, may these Design Principles help guide us to the correct decision.

#### Build for the consumer

APIs do not exist for their own sake, and are almost never the entry point for end users. Be mindful of which applications will be consuming our services, and go out of your way to cater to their requirements.

#### Convention over configuration

Our APIs should be intuitive, and should adhere to and implement community standards wherever possible. Problem-solving is at the core of development, and is also the great expense. Avoid implementing custom solutions to problems others have already solved, and in doing so we'll avoid creating new problems of our own design for API consumers and code base maintainers.

#### Optimize close to the metal

From authorization and serialization to filtering and sorting, we will gain greater efficiencies the closer we are to the database. Keep this in mind when building multi-layer and service-dependent APIs.

## 4. Interpreting the guidelines

### 4.1. Application of the guidelines

These guidelines are applicable to any REST API developed by Fourth.
Private or internal APIs SHOULD also try to follow these guidelines because internal services tend to eventually be exposed publicly.
 Consistency is valuable to not only external customers but also internal service consumers, and these guidelines offer best practices useful for any service.

There are legitimate reasons for exemption from these guidelines.
Obviously, a REST service that implements or must interoperate with some externally defined REST API must be compatible with that API and not necessarily these guidelines.
Some services MAY also have special performance needs that require a different format, such as a binary protocol.

### 4.2. Guidelines for existing services and versioning of services

We do not recommend making a breaking change to a service that pre-dates these guidelines simply for compliance sake.
The service SHOULD try to become compliant at the next version release when compatibility is being broken anyway.
When a service adds a new API, that API SHOULD be consistent with the other APIs

### 4.3. Requirements language

The keywords "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC 2119](https://www.ietf.org/rfc/rfc2119.txt).

## 5. Archetypes

When modeling an API’s resources, we can start with the some basic resource archetypes. Like design patterns, the resource archetypes help us consistently communicate the structures and behaviors that are commonly found in REST API designs. A REST API is composed of four distinct resource archetypes: docroot, resource, collection, and controller.

Each of these resource archetypes is described in the subsections that follow.

## Docroot

A REST API’s root resource is commonly referred to as a docroot, also known as a "reference document", and is typically the advertised entry point to an API.

The example URI below identifies the docroot for a contrived Soccer REST API:

    https://api.soccer.restapi.org

When determining an API’s URL structure, it is helpful to consider that all of its resources exist in a single reference document in which each resource is addressable at a unique path. Resources are grouped by type at the top level of this document. Individual resources are keyed by ID within these typed collections. Attributes and links within individual resources are uniquely addressable according to the resource object structure described above.

This concept of a reference document is used to determine appropriate URLs for resources as well as their relationships. It is important to understand that this reference document differs slightly in structure from documents used to transport resources due to different goals and constraints. For instance, collections in the reference document are represented as sets because members must be addressable by ID, while collections are represented as arrays in transport documents because order is significant.

### Resource
A resource is a singular concept that is akin to an object instance or database record. A resource's state representation typically includes both fields with values and links to other related resources. With its fundamental field and link-based structure, the resource type is the conceptual base archetype of the other resource archetypes. In other words, the three other resource archetypes can be viewed as specializations of the resource archetype.

Each URI below identifies a resource:

    https://api.soccer.restapi.org/alerts/245743
    https://api.soccer.restapi.org/leagues/seattle
    https://api.soccer.restapi.org/users/1234

A document may have child resources, also known as nested resources, which represent its specific subordinate concepts.

Each URI below identifies a nested resource.

    https://api.soccer.restapi.org/leagues/seattle/sponsors/microsoft
    https://api.soccer.restapi.org/leagues/seattle/teams/140
    https://api.soccer.restapi.org/users/1234/favorites/alonso

### Collection

A collection is a server-managed directory of resources. Clients may propose new resources to be added to a collection. However, it is up to the collection to choose to create a new resource, or not. A collection resource chooses what it wants to contain and also decides the URIs of each contained resource. Collections may also be nested.

Each URI below identifies a collection resource:

    https://api.soccer.restapi.org/leagues
    https://api.soccer.restapi.org/leagues/seattle/sponsors
    https://api.soccer.restapi.org/leagues/11/teams

### Controller

A controller models a procedural concept. Controllers are like executable functions, with parameters and return values; inputs and outputs.

Like a traditional web application’s use of HTML forms, a REST API relies on controllers to perform application-specific actions that cannot be logically mapped to one of the standard methods (create, retrieve, update, and delete, commonly referred to as "CRUD"). See the Methods section below for more details.

Controller names typically appear as the last segment in a URI path, with no child resources to follow them in the hierarchy. The example below shows a controller resource that allows a client to resend an alert to a user:

    POST /alerts/245743/resend

*Note:* Controller endpoints are exceptionally rare in modern RESTful APIs. If we find ourselves implementing more than a couple of controller endpoints, it may be worth rethinking our API design.

## 5. Structures

### Dates and Times

Dates MUST use the ISO8601 literal format: YYYY-MM-DDTHH:mm:ss.sssZ (Note the “Z” at the end of the string.)
At the very least, if you don’t want to use UTC time, please ensure to add a correct time offset to their timestamps (watch out for seasonal daylight savings time changes): 2017-04-15T13:40:00+02:00

Further reading on the reasoning for selecting this format: [Managing DateTime](https://www.codit.eu/blog/5-ways-to-better-manage-datetime-in-iot-solutions/)

## 6. Error Handling

From the perspective of the developer consuming the Fourth API, everything at the other side of that interface is a black box. Errors therefore become a key tool providing context and visibility into how to use an API.Error messages should be descriptive, corrective and informative.

### 6.1 Return an HTTP status code that closely matches the error condition

All client-side errors must be returned using the correct 4xx error code. In most cases, the API returns a 400 Bad Request. When a more specific error condition happens, the API returns a finer-grained 4xx status code that indicates the error condition. Examples of such codes include 409 (when you're trying to create a folder with a name that is already used in the current parent folder) or a 413 (when the request body is too large). HTTP-aware software can then interpret the error condition and handle it the right way.

### 6.2 Return human-readable error messages

APIs are meant to be consumed by developers (and apps). While it's essential to get HTTP status codes right, they are fairly coarse-grained and do not provide enough information to the developer. Consider the case where a developer tries to upload a file and gets a 400-status code: Was the request body malformed? Was the parent folder ID missing? Having an error message immediately makes it clear what went wrong. Ensure that internal-only messages (such as database version or stack traces) are excluded. Toget bonus points, localize the error messages.

### 6.3 Return machine-readable error codes

These are constants to indicate the error that happened. Usually they are exposed as strings (for example, 'folder_name_already_used' ) or integers. An app can consume the error codes (e.g., in a switch statement) to display dialogues that are relevant to the error condition. For instance, if the 'folder_name_already_used' error code is detected, the app can display a textbox with a "Rename" button that lets the client pick a different name for the folder.

### 6.4 Return the Invalid parameter name

Whenever possible, return the specific input parameter that the developer specified (or did not specify) that caused the API to choke on.

### 6.5 Set Location

This information is very useful for the developer to quickly identify the place that holds the erroneous piece of request. For REST APIs, location can be a URL, header or entity-body.

### 6.6 Provide a Help URL

If additional information to debug the error is available, include the link in the error response. The link destination, usually to a knowledge base, will provide more information about the error, often with suggested tips to resolve it. You might consider using the Link header with an extended relation type such as 'API-help' to communicate the help URL (since the RFC does not define a standard relation type for errors). Set any other response headers, if applicable.

#### Example JSON Error Response:

```JSON
{
  "statusCode": 403,
  "message": "The requesting user is not authorised to perform this functionality.",
  "errorCode": "user_not_authorised",
  "location": "URL",
  "helpUrl": "http://fourth-marketplace-purchasing-api-dv.azurewebsites.net/api/help/user_not_authorised"
}
```

## 7. Swagger API Documentation

All of our APIs MUST produce swagger documentation

This swagger documentation must be accessable via Layer 7

## 8. Layer 7

All of our APIs must be protected by Layer 7 and production environments MUST not be accessable by any other route other than Layer 7.

## 9. Collections

Resources may be displayed as a collection of resources.

Resource collections MUST be paged.  Default page size should be 100 resources per page, but this should be evaluated on a per-resource collection basis.

Resource collections SHOULD allow for filtering and sorting

### Collection Query Syntax

Filtering, Sorting and Paging on resource collections MUST use OData syntax.

### Collection Response Structure

The response should return an array of elements
The response MUST, at the very least, include a link element with a "rel"="next" if this is not the last page in the collection.

## 10. URL design

REST is resource-oriented and each resource is represented by a URL e.g. /orders/

A resource can be a singleton or a collection.

We can identify "orders" collection resource using the URI "/orders". We can identify a single "order" resource using the URI "/orders/{orderId}".

REST APIs use [Uniform Resource Identifiers (URIs)](https://en.wikipedia.org/wiki/Uniform_Resource_Identifier) to address resources. REST API designers should create URIs that convey a REST API’s resource model to its potential client developers. When resources are named well, an API is intuitive and easy to use. If done poorly, that same API can feel difficult to use and understand.

The constraint of uniform interface is partially addressed by the combination of URIs and HTTP verbs and using them in line with the standards and conventions.

Below are a few tips to get you going when creating the resource URIs for your new API.

### Nouns

RESTful URI should refer to a resource that is a thing (noun) instead of referring to an action (verb).

### Verbs

An *Endpoint* is represented by a Verb and URL e.g. GET: /orders

At a high level, HTTP verbs map to CRUD operations:

* [POST](https://tools.ietf.org/html/rfc7231#section-4.3.3) maps to Create
* [GET](https://tools.ietf.org/html/rfc7231#section-4.3.1) maps to Read
* [PUT](https://tools.ietf.org/html/rfc7231#section-4.3.4) and [PATCH](https://en.wikipedia.org/wiki/Patch_verb) map to Update
* [DELETE](https://tools.ietf.org/html/rfc7231#section-4.3.5) maps to Delete

Avoid using verbs in the URL.  HTTP verbs should be sufficient to describe the action being performed on the resource.

    *Don't do this:*
    POST: /orders/createNewOrder/

    *Rather do this:*
    POST: /orders/

### Plural resource nouns

When designing resources, the parent resource will be the collection and the child resource will be the individual resource.  

Always ensure that collection resources are plural.

    The parent resource is a collection:
    /orders

    The child resource is an individual resource:
    /orders/178273

### Nesting resources

Try to keep all resources accessable from the root resource if possible.

    /baskets
    /baskets/657
    /products
    /products/99565

You can supply alternate nested resources to represent hierarchical relationships too, but these should be secondary to the main resource URI

    /basket/657/products

### Grammar

Do not use trailing forward slash (/) in URIs

User Hyphens (-) to improve the readability of URIs in long path segments

Do not use underscores (_)
Depending on the application’s font, it’s possible that the underscore (_) character can either get partially obscured or completely hidden in some browsers or screens.

Do not use file extensions.  Use the Content-Type header to specify the file type instead:

    BAD:
    https://api.fourth.com/myapi/myresource.xml

    GOOD:
    https://api.fourth.com/myapi/myresource setting the Content-Type header to "application/xml"

## 11. HTTP Response Codes

A response's status is specified by its [HTTP status code](https://www.restapitutorial.com/httpstatuscodes.html): 

* 1xx for information,
* 2xx for success,
* 3xx for redirection,
* 4xx for client errors and
* 5xx for server errors.
