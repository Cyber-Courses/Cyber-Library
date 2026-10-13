---
title: "CouchDB injection"
order: 2
description: "Apache CouchDB is an HTTP/JSON document database whose design documents run server-side JavaScript, so injection targets those functions and the query, view, and configuration API."
keywords:
  - CouchDB injection
  - design document
  - query server
  - MapReduce
  - show and list functions
  - view query injection
---

# CouchDB

Apache CouchDB is a document database with an HTTP REST API and JSON documents. Every operation is an HTTP verb against a URL: `GET /db/doc`, `PUT /db/_design/app`, `POST /db/_find`. What makes CouchDB a distinct injection target is that its design documents carry **JavaScript** that runs server-side in a query server process: map and reduce functions, `show` and `list` transforms, `update` handlers, and `validate_doc_update` filters.

Injection here takes two shapes. The first targets the JavaScript surface: when an attacker can write a design document, or reach the `_config`/`query_servers` configuration, attacker-controlled code executes inside the server. The second targets the query and view API: key, range, and `include_docs` parameters passed into `_all_docs`, `_find`, and `_view` requests can be manipulated to read documents outside the intended scope.

## Pages

- **[JavaScript injection](javascript.md)**: Writing a design document or reaching the _config query_servers settings runs attacker JavaScript in CouchDB's query server process, reaching code execution.
- **[MapReduce injection](mapreduce.md)**: CouchDB map and reduce functions are JavaScript, so an attacker-controlled view definition executes server-side and emit() can leak documents outside the int...
- **[Show and list functions](show-list-function.md)**: CouchDB show and list functions transform documents with server-side JavaScript, so an injected function executes during rendering and can read beyond intend...
- **[View query injection](view-query.md)**: Manipulating key, startkey, endkey, include_docs, and _all_docs parameters on CouchDB view queries reads documents outside the intended key range.

## Tools

- **curl**: send raw HTTP/JSON requests to the CouchDB REST API.
- **Burp Suite**: intercept and tamper with document and view requests.

## References

- [Apache CouchDB: HTTP API reference](https://docs.couchdb.org/en/stable/api/index.html)
- [PayloadsAllTheThings: NoSQL Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/NoSQL%20Injection)
