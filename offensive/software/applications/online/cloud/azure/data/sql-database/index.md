---
title: "SQL Database"
order: 1
description: "Attacking Azure SQL Database: Entra and SQL authentication, firewall reach, contained users, and the server's managed identity."
keywords:
  - Azure SQL
  - SQL Database
  - contained users
  - firewall
  - managed identity
---

# SQL Database

Azure SQL Database is a managed SQL Server reached over TCP 1433, fronted by a server-level firewall and authenticated with either SQL logins or Microsoft Entra tokens. The offensive questions are whether the firewall lets you reach it, which credential or token authenticates you, and whether the logical server carries a **managed identity** you can drive from inside the database.

## Pages

- **[Access](access.md)**: reaching the database through Entra authentication, SQL logins, contained users, or the server managed identity.

## References

- [HackTricks Cloud: Azure SQL](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/index.html)
- [Microsoft: Azure SQL authentication](https://learn.microsoft.com/azure/azure-sql/database/authentication-aad-overview)
- [NetSPI: MicroBurst](https://github.com/NetSPI/MicroBurst)
