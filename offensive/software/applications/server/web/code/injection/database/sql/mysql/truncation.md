---
title: "SQL truncation attacks in MySQL"
description: "Abusing MySQL string truncation and trailing-space trimming to collide with an existing account such as admin, when strict SQL mode is off."
keywords:
  - SQL truncation attack
  - column length truncation
  - trailing space
  - username collision
  - account takeover
---

# Truncation

A truncation attack abuses how MySQL stored an oversized string in a fixed-length column. When strict SQL mode is off, inserting a value longer than the column silently truncates it to the column width instead of raising an error, and `CHAR`/`VARCHAR` comparisons ignore trailing spaces. Together these let a crafted value collide with an existing row after storage.

The classic target is registration against a `username VARCHAR(n)` column. Registering a name made of the victim's name, enough spaces to pass the column width, and a trailing character stores a truncated value equal to the victim's name:

```
username = "admin                 x"   (padded beyond VARCHAR length)
```

If the application first checks "does `admin` already exist?" against the full submitted string (which differs) and then inserts it, MySQL truncates and trims it back to `admin`, producing a second row that logs in as the administrator with a password the attacker set.

Two preconditions must both hold. First, a non-strict `sql_mode`: since MySQL 5.7 strict mode is on by default and turns the oversize insert into an error rather than truncating, so the attack applies to older servers or ones reconfigured to a permissive mode (confirm with `SELECT @@sql_mode`). Second, a `PAD SPACE` collation, so the stored trailing spaces are ignored on comparison and the padded value matches the plain name at login. This was the norm in older versions, but the MySQL 8.0 default collations (`utf8mb4_0900_ai_ci` and the other `utf8mb4_0900_*` set) are `NO PAD`, which makes trailing spaces significant and breaks the collision, so a column on a `PAD SPACE` collation such as `utf8mb4_general_ci` is also required.

## Tools

- Manual testing with Burp Repeater and the mysql client.

## References

- MySQL Reference Manual: `sql_mode`, string column storage and comparison
- OWASP Testing Guide: Testing for SQL Injection
