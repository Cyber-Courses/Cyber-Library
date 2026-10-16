---
title: "Zero trust : vérifier chaque requête, ne faire confiance à aucun emplacement"
description: "Le zero trust est un modèle de sécurité qui ne considère aucun emplacement réseau ni aucune identité comme implicitement fiable et qui vérifie chaque requête avant d'accorder l'accès."
keywords:
  - zero trust
  - architecture zero trust
  - ne jamais faire confiance toujours vérifier
  - point d'application des politiques
  - microsegmentation
  - vérification continue
---

# Zero trust

Le **zero trust** est un modèle de sécurité qui n'accorde aucune confiance automatique en fonction de la provenance d'une requête. Se trouver à l'intérieur du réseau de l'entreprise, sur un ordinateur portable professionnel ou derrière le pare-feu ne vaut rien en soi. Chaque tentative d'accès à une ressource est traitée comme si elle provenait d'un réseau ouvert et hostile et doit prouver son identité, l'état de son appareil et son droit à cette ressource précise avant que l'accès ne soit autorisé. La formule qui le résume est "ne jamais faire confiance, toujours vérifier".

## Le modèle de périmètre qu'il a remplacé

Pendant des décennies, la sécurité des entreprises a fonctionné comme un château entouré de douves. Un pare-feu gardait la frontière, tout ce qui se trouvait à l'extérieur était hostile et tout ce qui se trouvait à l'intérieur était considéré comme sûr. Le défaut était structurel : une fois qu'un attaquant franchissait les douves avec un mot de passe volé ou un ordinateur portable infecté, il héritait de la confiance du réseau et pouvait se déplacer latéralement vers presque tout.

Le télétravail, les services cloud et les appareils personnels ont ensuite entièrement dissous les douves, car les personnes et les données ne se trouvaient plus dans un seul bâtiment. Le terme "zero trust" a été popularisé par John Kindervag, analyste chez Forrester, vers 2010, et le projet BeyondCorp de Google, publié à partir de 2014, l'a prouvé à grande échelle en déplaçant les décisions d'accès du réseau vers les utilisateurs et les appareils individuels.

## Les trois composants logiques du NIST

La publication spéciale NIST 800-207 décrit l'architecture zero trust comme une séparation entre un plan de contrôle qui décide et un plan de données qui applique. Trois composants logiques portent le modèle :

| Composant | Rôle |
|---|---|
| Moteur de politiques | Pèse l'identité, l'état de l'appareil et le contexte, puis décide d'accorder, de refuser ou de révoquer l'accès |
| Administrateur de politiques | Exécute la décision en ouvrant ou en fermant le chemin de communication |
| Point d'application des politiques | Se place devant la ressource et ne laisse rien passer tant que le chemin n'est pas ouvert |

Le sujet n'est pas une passerelle unique mais cette boucle de décision, exécutée pour chaque requête plutôt qu'une seule fois à la connexion.

## Vérifier une requête unique

Suivons une ingénieure qui ouvre un tableau de bord interne de paie depuis un café.

1. La requête atteint un point d'application des politiques, jamais l'application directement. Se trouver sur un réseau particulier n'accorde rien.
2. L'identité est prouvée, généralement par une authentification multifacteur, en s'appuyant sur l'[authentification et autorisation](authentication-and-authorization.md).
3. L'appareil est vérifié : est-il géré, à jour de ses correctifs et exempt de compromission connue ?
4. Le moteur de politiques évalue l'ensemble du contexte et accorde l'accès à cette seule ressource, pour cette seule session, au niveau minimal nécessaire, en appliquant le [moindre privilège](least-privilege.md).
5. La session est réévaluée en continu. Si l'appareil cesse d'être conforme ou si le comportement devient anormal, l'accès est révoqué en cours de session.

## Le prix de la suppression de la confiance implicite

Le zero trust n'est pas gratuit. Chaque requête porte désormais le coût d'une consultation de politique, ce qui ajoute de la latence et fait du moteur de politiques une dépendance critique : s'il est lent ou hors ligne, le travail légitime s'arrête. Des signaux d'identité et d'appareil robustes doivent exister et rester exacts, sinon les décisions sont prises sur de mauvaises données. Mal mené, les points d'application deviennent un nouveau périmètre déguisé.

Il ne remplace pas non plus les autres défenses. Restreindre chaque session à une seule ressource réduit la [surface d'attaque](../concepts/assets-and-attack-surface.md) accessible, et le zero trust se combine avec la [défense en profondeur](defense-in-depth.md) au lieu de s'y substituer, tandis que des journaux détaillés par requête alimentent la [responsabilité](accountability.md).

## Références

- [NIST SP 800-207, Zero Trust Architecture](https://csrc.nist.gov/pubs/sp/800/207/final)
- [NIST SP 1800-35, Implementing a Zero Trust Architecture](https://www.nccoe.nist.gov/projects/implementing-zero-trust-architecture)
- [CISA Zero Trust Maturity Model](https://www.cisa.gov/zero-trust-maturity-model)
- [BeyondCorp: A New Approach to Enterprise Security](https://research.google/pubs/pub43231/)
