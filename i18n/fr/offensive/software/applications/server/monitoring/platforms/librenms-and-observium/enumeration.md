---
title: "Énumération : identifier LibreNMS et Observium"
description: "LibreNMS et Observium s'identifient à partir de l'image de marque et de la version de leur interface web, exposées dans la page de connexion et l'API. La version décide quelles failles d'injection de commandes authentifiées s'appliquent, et distinguer LibreNMS d'Observium importe car leurs fonctionnalités et leurs historiques de vulnérabilités diffèrent malgré l'origine commune."
keywords:
  - librenms enumeration
  - observium
  - version
  - fingerprint
  - api
---

# Énumération

Identifiez lequel des deux s'exécute et sa version. LibreNMS et Observium partagent une lignée mais ont divergé, de sorte que leurs fonctionnalités, leur API et leurs vulnérabilités diffèrent, et l'image de marque de l'interface web, la structure de la page et la chaîne de version les distinguent. LibreNMS expose une version dans l'UI et son API ; Observium de même. La version est le fait qui décide quelles failles d'injection de commandes authentifiées et autres s'appliquent à la cible, lisez-la donc avant d'attaquer.

```bash
# distinguish the product and read the version
curl -sk https://<target>/ | grep -ioE 'LibreNMS|Observium'
curl -sk https://<target>/api/v0/system -H 'X-Auth-Token: <token>'   # LibreNMS API (version/system)
curl -sk https://<target>/ | grep -ioE 'version[^<]*[0-9][0-9.]+'
```

## Notes d'exploitation

- Distinguez d'abord LibreNMS d'Observium : leurs historiques de vulnérabilités et leurs fonctionnalités diffèrent malgré l'origine commune, de sorte que le produit sélectionne les voies d'[injection de commandes](command-injection.md) applicables.
- La version correspond aux failles de RCE authentifiées connues ; identifiez-la précisément.
- LibreNMS possède une API basée sur des jetons qui expose les données système et d'équipements une fois authentifié ; notez-le pour l'énumération après l'accès.
- Orientez vers l'[authentification](authentication.md), puis vers l'injection de commandes et la [récolte d'identifiants](credential-harvesting.md).

## Références

- [LibreNMS API](https://docs.librenms.org/API/)
- [Observium documentation](https://docs.observium.org/)
