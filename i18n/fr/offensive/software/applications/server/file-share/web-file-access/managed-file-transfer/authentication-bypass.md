---
title: "Contournement d'authentification : atteindre les fonctions MFT sans identifiants"
description: "Les appliances MFT ont livré à plusieurs reprises des contournements d'authentification : endpoints d'API ou de servlet non authentifiés, gestion défaillante des sessions ou des jetons, et erreurs de logique permettant à un attaquant de créer ou d'assumer un compte administrateur. Contourner l'authentification expose l'accès aux fichiers et les fonctions d'administration, qui à eux seuls font fuir des données et s'enchaînent souvent en exécution de code à distance complète."
keywords:
  - authentication bypass
  - mft
  - admin account
  - unauthenticated endpoint
  - session
---

# Contournement d'authentification

Une faiblesse récurrente du MFT consiste à atteindre une fonctionnalité protégée sans identifiants valides. Les formes varient : un endpoint d'API ou de servlet qui effectue des actions sensibles sans vérification d'authentification, une gestion des sessions ou des jetons pouvant être forgée ou rejouée, et des failles de logique permettant à une requête non authentifiée de créer un administrateur ou d'en assumer un existant. Comme les portails MFT frontalisent à la fois l'accès aux fichiers et l'administration, un contournement expose immédiatement les données des partenaires et les contrôles d'administration, et la surface d'administration s'enchaîne alors fréquemment en exécution de code.

```bash
# probe for unauthenticated access to admin/API endpoints (product-specific paths)
curl -sk https://<target>/api/v1/...           # does a sensitive API answer without auth?
curl -sk https://<target>/<admin-servlet>      # admin function reachable unauthenticated?
# account-creation/assume bypass: craft the request the flaw permits, then use the session
curl -sk -X POST https://<target>/<endpoint> -d '<crafted body>' -i   # creates/assumes admin
```

## Notes d'exploitation

- L'appliance et la version déterminent l'endpoint et la requête exacts ; dressez d'abord l'empreinte, puis visez le contournement connu pour ce build. Beaucoup sont entièrement non authentifiés, de sorte que l'accessibilité est la seule condition préalable.
- Un contournement de création ou d'assomption de compte administrateur est le plus puissant, il livre l'intégralité du portail d'administration, d'où découlent l'accès aux fichiers et les voies d'injection/RCE.
- Même un contournement non administrateur qui n'atteint que les endpoints de listage ou de téléchargement de fichiers est significatif pour le MFT, car l'intérêt même de l'appliance réside dans les fichiers sensibles qu'elle détient.
- Enchaînez avec [Injection vers RCE](injection-to-rce.md) : l'accès administrateur expose souvent la configuration ou la fonctionnalité où une faille d'injection donne une exécution de code ; voir les [campagnes spécifiques](known-mft-exploits/index.md).

## Références

- [CISA : bulletins sur l'exploitation du MFT](https://www.cisa.gov/news-events/cybersecurity-advisories)
- [OWASP : authentification défaillante](https://owasp.org/www-project-top-ten/)
