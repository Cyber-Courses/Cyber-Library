---
title: "Énumération : identification de PRTG"
order: 4
description: "PRTG s'identifie par son interface web, sa page de connexion et sa chaîne de version, exposées sur les ports du serveur central. La version importe car la faiblesse d'injection de commandes de notification et certaines vulnérabilités dépendent de la version, de sorte que lire le build depuis la page de connexion ou l'API oriente l'attaque."
keywords:
  - prtg enumeration
  - version
  - login page
  - fingerprint
  - paessler
---

# Énumération

Identifier PRTG et sa version est rapide et oriente l'attaque. L'interface web (le serveur central PRTG, par défaut 8080/8443, souvent présentée sur 80/443) possède une page de connexion et une image de marque reconnaissables, et la chaîne de version apparaît dans la page, les ressources et les réponses de l'API. La version importe car l'exécution de code basée sur les notifications et les vulnérabilités connues spécifiques dépendent du build et de la présence ou non des correctifs pertinents. Lire la version depuis la page de connexion ou l'API est la première étape avant de s'authentifier.

```bash
# fingerprint PRTG and its version
curl -sk https://<target>/index.htm | grep -ioE 'PRTG[^<]*'
curl -sk 'https://<target>/api/status.json?id=0' | grep -i version   # API status (version)
curl -sk https://<target>/login.htm | grep -ioE 'Version [0-9.]+'
```

## Notes d'exploitation

- La chaîne de version issue de la page de connexion ou de `api/status.json` indique si le correctif d'injection de commandes de notification et d'autres correctifs sont présents, ce qui oriente l'[abus des notifications](notification-command-abuse.md) et les [exploits connus](known-exploits.md).
- PRTG s'exécute sous Windows avec le service central en tant que Local System, de sorte qu'une exécution réussie équivaut à SYSTEM ; cela justifie de confirmer le produit, car l'impact est élevé.
- L'interface peut se trouver sur des ports non standard (8080/8443) même lorsque 80/443 sont présentés ; scannez à la recherche du serveur central.
- Dirigez-vous vers l'[authentification](authentication.md) ; le compte `prtgadmin` par défaut est le premier gain courant.

## Références

- [Documentation de l'API PRTG](https://www.paessler.com/manuals/prtg/application_programming_interface_api_definition)
- [HackTricks : PRTG](https://book.hacktricks.xyz/network-services-pentesting/pentesting-web/prtg)
