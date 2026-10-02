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

The precondition is a non-strict `sql_mode`. Since MySQL 5.7 strict mode is on by default, which turns the oversize insert into an error and closes the attack, so it applies to older servers or ones reconfigured to a permissive mode. Confirm with `SELECT @@sql_mode` where you can reach it.

## References

- MySQL Reference Manual: `sql_mode`, string column storage and comparison
- OWASP Testing Guide: Testing for SQL Injection
