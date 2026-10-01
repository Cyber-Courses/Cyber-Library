---
title: "CouchDB injection"
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
