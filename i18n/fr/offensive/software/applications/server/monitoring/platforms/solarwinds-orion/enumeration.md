---
title: "Énumération : identification de SolarWinds Orion"
order: 4
description: "Orion s'identifie par sa console web (Orion/Login.aspx), l'image de marque des pages et la version présente dans l'interface et les ressources. La version et les modules installés déterminent quelles chaînes de contournement et de RCE connues s'appliquent, de sorte que lire le build avant d'attaquer oriente le choix entre un exploit non authentifié et la voie des identifiants."
keywords:
  - solarwinds enumeration
  - orion login
  - version
  - modules
  - fingerprint
---

# Énumération

SolarWinds Orion expose une console web reconnaissable (la page de connexion à `/Orion/Login.aspx`) avec l'image de marque SolarWinds, et la version apparaît dans l'interface, le pied de page et les ressources statiques. La version et les modules Orion installés (Network Performance Monitor, Server & Application Monitor et d'autres) déterminent quelles vulnérabilités connues s'appliquent, puisque les graves chaînes de contournement et d'exécution de code à distance sont propres à la version et au module. Lire d'abord le build détermine si un exploit non authentifié est disponible pour cette instance ou si la voie des identifiants est à privilégier.

```bash
# fingerprint the console and version
curl -sk https://<target>/Orion/Login.aspx | grep -ioE 'SolarWinds|Orion Platform [0-9.]+'
curl -sk https://<target>/Orion/ | grep -ioE 'version[^<]*[0-9][0-9.]+'
# the Orion API/service endpoints (e.g. /SolarWinds/InformationService/) indicate modules
```

## Notes d'exploitation

- La version de la plateforme Orion et les modules installés correspondent aux [exploits connus](known-exploits.md) applicables ; identifiez les deux, car les chaînes sont propres à la version et au module.
- Une console accessible et une version dans la plage visée peuvent constituer un exploit non authentifié ; sinon, les voies de l'[authentification](authentication.md) et de la [récolte d'identifiants](credential-harvesting.md) s'appliquent.
- Orion est souvent partiellement exposé sur Internet (console web ou points de terminaison d'API) ; identifiez quels composants sont accessibles.
- Dirigez-vous vers les chaînes d'exploitation ou vers l'authentification selon l'accès et la version.

## Références

- [Plateforme SolarWinds Orion](https://www.solarwinds.com/)
- [Avis de sécurité SolarWinds](https://www.solarwinds.com/trust-center/security-advisories)
