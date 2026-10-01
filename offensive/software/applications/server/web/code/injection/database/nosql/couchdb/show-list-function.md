---
title: "Show and list function injection in CouchDB"
description: "CouchDB show and list functions transform documents with server-side JavaScript, so an injected function executes during rendering and can read beyond intended data."
keywords:
  - CouchDB show function
  - CouchDB list function
  - server-side rendering
  - design document
  - provides
  - getRow
---

# Show and list functions

CouchDB can render documents into arbitrary output formats server-side. A `show` function transforms a single document into an HTTP response; a `list` function transforms the rows of a view into one. Both are JavaScript stored in a design document and executed by the query server at request time. Controlling either function, or the design document that holds it, runs attacker JavaScript during rendering.

## Show functions

A `show` function takes a document and the request object and returns a response body:

```json
{
  "shows": {
    "render": "function(doc, req){ return { body: JSON.stringify(doc), headers: { 'Content-Type': 'application/json' } }; }"
  }
}
```

It is invoked against a document ID:

```http
GET /app/_design/pub/_show/render/user:42 HTTP/1.1
```

An injected `show` runs with the query server's reach. Because the function receives the whole `doc`, it can serialize fields the application never meant to publish, and because it also receives `req`, it can branch on attacker-supplied query parameters:

```javascript
function(doc, req){
  // req.query controlled by the caller
  return { body: JSON.stringify(doc[req.query.field]) };
}
```

A request such as `_show/render/user:42?field=password_hash` then returns the targeted field. The `req` object also exposes headers, cookies, and the raw path, all attacker-influenced input that an injected function can act on.

## List functions

A `list` function runs over a view's rows, pulling them with `getRow()` and streaming output:

```json
{
  "lists": {
    "dump": "function(head, req){ var row; while(row = getRow()){ send(JSON.stringify(row.value)); } }"
  }
}
```

It is invoked as a pair of design document, list name, and view name:

```http
GET /app/_design/pub/_list/dump/leak HTTP/1.1
```

Here `leak` is a view in the same design document. An injected `list` walks every row the view produced and sends each value to the caller, bypassing any pagination or field selection the application layered on top. Pairing a permissive list with a broad view (see [MapReduce injection](mapreduce.md)) streams the full set in one response.

## The provides/registerType surface

Show and list functions commonly branch on the requested format via `provides()` and `registerType()`:

```javascript
function(doc, req){
  provides('html', function(){ return '<p>' + doc.name + '</p>'; });
  provides('json', function(){ return JSON.stringify(doc); });
}
```

An injected function can register a handler that emits attacker-chosen content, and because the callback bodies are plain JavaScript, they are a code sink like any other field. Where an application assembles a show/list body from user input (a template fragment, a field list), the attacker closes the surrounding expression and appends their own statements.

## Triggering and probing

Show and list execution is a simple `GET`, so no write is needed once the function exists. To confirm an injection point when an application builds these functions dynamically, submit input that breaks the surrounding JavaScript and watch for a query-server compilation error, or supply a benign branch (`provides('txt', function(){ return 'marker'; })`) and request that format to see the injected code run. An injected function that reads `req` turns the render endpoint into a parameter-driven read primitive over documents the caller can name.

## References

- [Apache CouchDB: Show functions](https://docs.couchdb.org/en/stable/ddocs/ddocs.html#show-functions)
- [Apache CouchDB: List functions](https://docs.couchdb.org/en/stable/ddocs/ddocs.html#list-functions)
- [PayloadsAllTheThings: NoSQL Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/NoSQL%20Injection)
