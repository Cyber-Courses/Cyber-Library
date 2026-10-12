---
title: "Détection de version : déterminer la version du client de bureau à distance"
description: "La version du client décide des attaques applicables : faisabilité du brute-force (les éditeurs ont ajouté la limitation de débit au fil du temps), récupérabilité des identifiants stockés (les formats de stockage ont changé) et vulnérabilités produit spécifiques auxquelles une build est exposée. La version se lit depuis l'interface du client, les fichiers installés, le registre ou la poignée de main, puis se rapproche des avis connus."
keywords:
  - détection de version
  - version du client
  - rapprochement avec les avis
  - teamviewer
  - anydesk
---

# Détection de version

La version est décisive pour ces produits parce que tant de choses ont changé d'une version à l'autre. Les éditeurs ont ajouté la limitation de débit des connexions et le verrouillage après des campagnes massives de compromission de comptes, de sorte que la faisabilité du [brute-force de mot de passe](../authentication/weak-passwords.md) en ligne dépend de la version. Le format et la récupérabilité des identifiants stockés (pour l'[accès sans surveillance](../authentication/unattended-access.md)) ont changé selon les versions. Et les vulnérabilités de contournement d'authentification et d'exécution de code propres à chaque produit ne s'appliquent qu'à des builds particulières. La version se lit depuis l'interface « À propos » du client, les métadonnées du binaire installé, le registre ou les données d'application, ou dans certains cas la poignée de main de la connexion, puis se rapproche des avis.

```bash
# host: read the client version from binary metadata / registry
(Get-Item 'C:\Program Files\TeamViewer\TeamViewer.exe').VersionInfo.ProductVersion   # PowerShell
reg query 'HKLM\SOFTWARE\WOW6432Node\TeamViewer' /v Version 2>nul
# AnyDesk version from its binary or app-data
# match the build to the vendor advisory for the applicable exploit
```

## Notes d'exploitation

- La version conditionne la faisabilité : une ancienne build peut autoriser un brute-force en ligne qu'une version actuelle limite en débit, et peut stocker les identifiants sans surveillance sous une forme récupérable que les versions récentes protègent.
- Rapprochez la build des avis produit pour les [exploits connus](../known-product-exploits/index.md) (contournement d'authentification, RCE) ; ceux-ci sont spécifiques à la version.
- Là où vous n'avez qu'un accès réseau, le comportement de la poignée de main ou du relais peut suggérer la version ; sur un hôte, les métadonnées du binaire et le registre la donnent exactement.
- Combinez avec la [détection de service](service-detection.md) pour compléter l'empreinte avant de choisir l'attaque.

## Références

- [TeamViewer release/security notes](https://www.teamviewer.com/en/trust-center/security/)
- [AnyDesk security advisories](https://anydesk.com/en/security)
