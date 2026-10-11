---
title: "Énumération : identifier Cacti"
order: 4
description: "Cacti s'identifie par son interface web sous /cacti/ et sa chaîne de version dans la page et le changelog, et la version est décisive car ses vulnérabilités d'exécution de code à distance sont spécifiques à la version. Lire le build exact oriente le choix de l'exploit, y compris la présence ou non d'une voie d'injection de commandes non authentifiée."
keywords:
  - cacti enumeration
  - version
  - fingerprint
  - cacti path
  - advisory
---

# Énumération

L'interface web de Cacti se trouve sous `/cacti/` et expose le produit et la version dans la page de connexion, le pied de page et le `CHANGELOG` fourni. La version est le fait décisif, car les vulnérabilités d'injection de commandes et d'injection SQL de Cacti, y compris celles non authentifiées, sont spécifiques à des builds particuliers. Lire d'abord la version exacte vous indique quelle chaîne d'exploitation s'applique et si une voie d'exécution de code à distance non authentifiée (telle que celle dans la gestion de l'agent distant) est présente sur cette instance.

```bash
curl -sk https://<target>/cacti/ | grep -ioE 'Version [0-9.]+'
curl -sk https://<target>/cacti/CHANGELOG | head                 # often readable, exact version
curl -sk https://<target>/cacti/include/cacti_version            # version file in some builds
```

## Notes d'exploitation

- La version correspond directement aux [exploits connus](known-exploits.md) applicables ; les bugs de RCE de Cacti sont spécifiques à la version, identifiez-la donc précisément (le `CHANGELOG` ou le fichier de version la donne exactement).
- Déterminez si une voie non authentifiée s'applique à ce build (certaines failles d'injection de commandes ne nécessitent aucune connexion), ce qui change le besoin d'[identifiants](authentication.md) au préalable.
- Le chemin `/cacti/` et le fichier `CHANGELOG`/version lisible rendent l'identification facile et non authentifiée.
- Orientez vers les chaînes d'exploitation ou, avec un accès, vers la [récolte d'identifiants](credential-harvesting.md).

## Références

- [Cacti releases](https://github.com/Cacti/cacti/releases)
- [Cacti security advisories](https://github.com/Cacti/cacti/security/advisories)
