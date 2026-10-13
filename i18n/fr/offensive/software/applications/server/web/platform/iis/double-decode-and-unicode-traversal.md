---
order: 2
title: "Double décodage et traversée Unicode IIS : décodage multi-étapes qui fait passer en douce ../"
description: "Exploitation du comportement de décodage d'IIS : traversée double-encodée qui se résout en ../ après un second décodage, et encodages Unicode trop longs du slash sur les IIS anciens, pour s'échapper de la racine web."
keywords:
  - double décodage
  - double encodage
  - traversée unicode
  - utf-8 trop long
  - IIS
---

# Double décodage et traversée Unicode

Historiquement, IIS décodait les chemins de requête plus d'une fois, ou décodait des formes Unicode permissives, de sorte qu'un payload qui semble inoffensif à un filtre devient une séquence de traversée après un décodage ultérieur. C'est le mécanisme derrière les bugs classiques de traversée de répertoire d'IIS, et le motif de double décodage apparaît encore partout où des étapes de décodage s'empilent (un WAF décode une fois, IIS décode à nouveau).

## Double encodage

`/` est `%2f` ; le `%` dans `%2f` est `%25`, donc `%252f` se décode une fois en `%2f` et deux fois en `/`. De même `%252e` se décode deux fois en `.`, donc `%252e%252e%252f` devient `../` après deux passes :

```
GET /scripts/%252e%252e/%252e%252e/winnt/system32/cmd.exe HTTP/1.1
GET /app/%252e%252e%252fweb.config HTTP/1.1
```

Un filtre qui décode une fois et cherche `../` voit `%2e%2e/` (aucune correspondance) et le transmet ; IIS décode à nouveau et mappe `../`.

## Unicode trop long (IIS ancien)

Les IIS plus anciens acceptaient des encodages UTF-8 trop longs de `/` et `\`, de sorte qu'un seul décodage d'une séquence non canonique produisait un séparateur que le filtre ne reconnaissait jamais :

```
GET /scripts/..%c0%af../winnt/system32/cmd.exe HTTP/1.1
GET /scripts/..%c1%9c../ HTTP/1.1
```

`%c0%af` et `%c1%9c` sont des encodages trop longs de `/` et `\`. Ceux-ci ciblent la traversée Unicode classique d'IIS et restent utiles à tester contre les stacks Windows anciens.

## Exploitation

- Prenez un payload de traversée filtré en forme simple-encodée et ré-encodez chaque `%` en `%25` : `..%2f` devient `..%252f` ; variez les profondeurs pour correspondre au nombre de passes de décodage dans la chaîne.
- Ajoutez des variantes de slash trop long/Unicode contre les IIS plus anciens.
- Ciblez des chemins Windows (`/windows/win.ini`, `web.config`, source de l'application) et, sur les stacks très anciens, `cmd.exe` pour l'exécution de commandes via le répertoire scripts.
- Envoyez avec `curl --path-as-is` pour que le client ne décode ni ne normalise le payload au préalable.

```bash
curl --path-as-is "https://target/app/%252e%252e%252fweb.config"
```

## Outils

- Burp Intruder avec des wordlists de double-encodage/Unicode ; **curl --path-as-is**.

## Références

- OWASP : Double encoding
- Avis historiques IIS sur l'Unicode et le double décodage
