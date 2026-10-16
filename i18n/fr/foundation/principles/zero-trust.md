---
title: "Zero Trust : vérifier chaque requête, ne faire confiance à aucun emplacement"
description: "Le modèle de sécurité qui ne considère aucun emplacement réseau ni aucune identité comme digne de confiance par défaut et qui vérifie chaque requête avant d'accorder l'accès."
keywords:
  - zero trust
  - architecture zero trust
  - ne jamais faire confiance toujours vérifier
  - vérification d'identité
  - microsegmentation
  - authentification continue
---

# Zero trust

**Le zero trust** est un modèle de sécurité fondé sur une règle unique : aucun utilisateur, aucun appareil et aucun emplacement réseau n'est digne de confiance par défaut, et chaque requête d'accès à une ressource doit être vérifiée avant d'être autorisée. Sa devise est "ne jamais faire confiance, toujours vérifier". Au lieu de se demander "cette connexion provient-elle de notre réseau interne ?", un système zero trust se demande "qui est-ce, sur quel appareil, que demande-t-il, et cela a-t-il du sens en ce moment ?", et il repose cette question à chaque requête.

## Le château fort et ses douves ne tiennent plus

La sécurité d'entreprise traditionnelle traçait une frontière nette autour du réseau de l'organisation. Un pare-feu gardait la bordure : tout ce qui se trouvait à l'intérieur du périmètre était considéré comme sûr, tout ce qui se trouvait à l'extérieur comme hostile. Une fois les douves franchies, un utilisateur ou une machine pouvait généralement atteindre la plupart des systèmes internes.

Ce modèle supposait que les personnes, les applications et les données se trouvaient toutes dans le même bâtiment. Le télétravail, les services cloud, les applications SaaS et les appareils mobiles ont rendu cette hypothèse caduque : les ressources sont désormais réparties entre bureaux, domiciles et plusieurs clouds, et un pare-feu en bordure ne peut pas protéger ce qui se trouve en dehors de lui. Le modèle échouait aussi selon ses propres critères, car un seul identifiant volé ou un seul ordinateur portable compromis à l'intérieur héritait de la confiance accordée à tout le réseau et pouvait y circuler librement.

L'idée de supprimer la confiance implicite est née en réaction. L'analyste de Forrester John Kindervag a popularisé le terme "zero trust" vers 2010, et le projet BeyondCorp de Google, décrit publiquement en 2014, en a démontré le fonctionnement à grande échelle : Google a déplacé les contrôles d'accès du périmètre réseau vers les utilisateurs et les appareils eux-mêmes, permettant aux employés de travailler depuis des réseaux non fiables sans VPN traditionnel.

## Anatomie d'une décision d'accès

La meilleure façon de comprendre le zero trust est de suivre une requête. Imaginons qu'une ingénieure ouvre le tableau de bord interne de la paie depuis un ordinateur portable, dans un café.

1. **La requête atteint un point d'application, pas l'application elle-même.** Un point d'application des politiques (une passerelle, un proxy ou un agent) se place devant l'application de paie. Rien n'atteint l'application directement, et le fait d'être sur un réseau particulier n'accorde rien.
2. **L'identité est prouvée.** L'ingénieure s'authentifie, généralement avec une authentification multifacteur. Le service qui va répondre, ainsi que tout service qu'il appelle, s'authentifie également.
3. **L'appareil est examiné.** S'agit-il d'un ordinateur géré ? Son système d'exploitation est-il à jour, son disque chiffré, sa protection des terminaux active ? Un utilisateur légitime sur un appareil compromis reste un risque.
4. **Le contexte est évalué.** L'emplacement, l'heure et le comportement sont comparés à ce qui est habituel pour cet utilisateur. Se connecter depuis deux continents en moins d'une heure, par exemple, augmente le score de risque.
5. **Le moteur de politiques décide, pour cette requête uniquement.** Il vérifie si cette identité, sur cet appareil, dans ce contexte, doit accéder à cette ressource précise. Les ingénieurs n'ont peut-être aucun besoin des données de paie, et en vertu du [moindre privilège](least-privilege.md), la réponse peut tout simplement être non.
6. **La confiance est réévaluée en permanence.** Si l'accès est accordé, il est limité à ce dont la requête a besoin, et la session est réévaluée à mesure que les conditions changent. Si l'appareil cesse d'être conforme ou si le comportement devient anormal en cours de session, l'accès peut être coupé.

Le NIST SP 800-207 nomme les composants logiques centraux de ce flux : le **moteur de politiques** (policy engine) prend la décision, l'**administrateur de politiques** (policy administrator) l'exécute en ouvrant ou en fermant le chemin, et le **point d'application des politiques** (policy enforcement point) se situe sur le chemin des données et autorise ou bloque effectivement le trafic. Les mécanismes sous-jacents des étapes 2 et 5 sont détaillés dans [authentification et autorisation](authentication-and-authorization.md).

## Les signaux derrière "vérifier"

"Vérifier" ne se limite pas à contrôler un mot de passe. Une politique zero trust combine plusieurs types d'éléments de preuve, chacun détectant ce que les autres manquent.

| Signal | Ce qui est vérifié | Ce qu'il détecte |
|---|---|---|
| Identité | Qui est l'utilisateur ou le service, prouvé par une authentification forte, généralement multifacteur | Des mots de passe devinés ou hameçonnés utilisés seuls |
| Posture de l'appareil | Niveau de correctifs, chiffrement, statut de gestion, protection des terminaux | Un utilisateur légitime sur une machine infectée ou non gérée |
| Contexte et comportement | Emplacement, heure, schémas de requêtes comparés à une référence | Des sessions et des identifiants volés utilisés de façon inhabituelle |
| Sensibilité de la ressource | Le degré de criticité des données ou du système demandés | Un accès à faible niveau d'assurance à des cibles de grande valeur |

Deux techniques structurelles donnent tout leur sens à ces contrôles. La **microsegmentation** divise le réseau en petites zones, de sorte qu'atteindre un système ne permet pas d'atteindre le reste : c'est ce qui empêche un attaquant qui parvient à entrer de trouver de nombreux endroits où aller. L'**évaluation continue** signifie que la confiance n'est jamais un tampon unique apposé à la connexion : elle est réévaluée tout au long de la session. Ensemble, elles font du zero trust une stratégie de confinement des brèches, en ajoutant des couches fondées sur l'identité par-dessus les contrôles réseau et hôte décrits dans [défense en profondeur](defense-in-depth.md).

## Des recommandations du NIST à une échéance fédérale

Le zero trust est passé du vocabulaire des fournisseurs à un cadre formel au travers d'une courte série de documents du gouvernement américain :

- **Août 2020 : NIST SP 800-207.** Le NIST a publié *Zero Trust Architecture*, qui définit le concept, en énonce les principes et décrit des modèles de déploiement. Il reste la définition de référence la plus citée.
- **Janvier 2022 : OMB M-22-09.** L'Office of Management and Budget a demandé aux agences fédérales civiles d'adopter une stratégie zero trust, avec des objectifs précis concernant les identités, les appareils, les réseaux, les applications et les données, à atteindre d'ici la fin de l'exercice budgétaire 2024.
- **Avril 2023 : CISA Zero Trust Maturity Model 2.0.** La CISA a organisé le travail en cinq piliers (identité, appareils, réseaux, applications et charges de travail, données) et trois capacités transversales (visibilité et analyse, automatisation et orchestration, gouvernance). Chaque pilier est évalué selon quatre stades : traditionnel, initial, avancé et optimal.

Le modèle de maturité dépasse le cadre gouvernemental, car il présente le zero trust comme un parcours mesuré pilier par pilier, et non comme un interrupteur qu'une organisation actionnerait. De nombreuses entreprises privées et d'autres gouvernements l'utilisent comme feuille de route.

## Là où les programmes zero trust s'enlisent

- **Acheter une boîte étiquetée "zero trust".** Le zero trust est une architecture et un état d'esprit, pas un produit unique que l'on achète et que l'on active. Les outils aident, mais seulement au sein d'une conception qui supprime partout la confiance implicite.
- **S'arrêter à la connexion.** Vérifier l'identité une seule fois puis faire confiance à toute la session réintroduit discrètement la confiance implicite que le modèle est censé supprimer.
- **Vérifier l'utilisateur mais pas l'appareil.** Une authentification forte depuis une machine compromise ou non mise à jour offre toujours une porte d'entrée à un attaquant.
- **Ajouter de la friction sans réfléchir.** Des contrôles qui interrompent sans cesse les utilisateurs les poussent vers des contournements, sapant la sécurité qu'ils étaient censés apporter. Les bonnes conceptions rendent les vérifications invisibles quand le risque est faible et ne se renforcent que lorsqu'il augmente.
- **Rafistoler au lieu de concevoir.** Les hypothèses de vérification préalable sont bien plus faciles à intégrer dans de nouveaux systèmes qu'à greffer sur des applications héritées qui supposent un réseau interne de confiance, ce qui explique pourquoi le zero trust s'associe naturellement à la [sécurité dès la conception](security-by-design.md).

## Références

- [NIST SP 800-207, Zero Trust Architecture](https://csrc.nist.gov/pubs/sp/800/207/final)
- [NIST Glossary: Zero Trust](https://csrc.nist.gov/glossary/term/zero_trust)
- [CISA: Zero Trust Maturity Model](https://www.cisa.gov/zero-trust-maturity-model)
- [OMB M-22-09, Moving the U.S. Government Toward Zero Trust Cybersecurity Principles](https://www.whitehouse.gov/wp-content/uploads/2022/01/M-22-09.pdf)
- [Google Research: BeyondCorp, A New Approach to Enterprise Security (2014)](https://research.google/pubs/beyondcorp-a-new-approach-to-enterprise-security/)
