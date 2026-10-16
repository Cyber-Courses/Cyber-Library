---
title: "Mesures et contre-mesures : comment les défenses réduisent le risque"
description: "Les protections que les organisations mettent en place pour prévenir, détecter et se rétablir après un incident de sécurité, regroupées en mesures préventives, détectives et correctives."
keywords:
  - mesures de sécurité
  - contre-mesures
  - mesures préventives
  - mesures détectives
  - mesures correctives
  - défense en profondeur
---

# Mesures et contre-mesures

**Une mesure** est toute protection ou disposition mise en place pour réduire le risque de sécurité, que ce soit en diminuant la probabilité qu'un événement fâcheux survienne ou en limitant les dégâts lorsqu'il se produit. Le terme contre-mesure est employé de façon presque interchangeable, même s'il met souvent l'accent sur une réponse visant une menace précise. Les mesures sont la réponse concrète au risque : une fois qu'une organisation sait quels [actifs](assets-and-attack-surface.md) elle protège et quelles [menaces](threats-and-threat-actors.md) pourraient les atteindre, les mesures sont les actions qu'elle entreprend pour faire pencher les probabilités en sa faveur.

## Avant, pendant, après : le moment où agit une mesure

La façon la plus courante de classer une mesure consiste à regarder à quel moment elle intervient dans le déroulement d'un événement de sécurité.

1. **Les mesures préventives agissent avant l'événement** et cherchent à l'empêcher purement et simplement. Les autorisations d'accès, l'authentification forte, le chiffrement, les pare-feu et la sensibilisation à la sécurité réduisent tous la probabilité qu'une attaque réussisse.
2. **Les mesures détectives agissent pendant qu'il se déroule** en révélant que quelque chose s'est produit ou est en train de se produire. La journalisation et la surveillance, les systèmes de détection d'intrusion, les audits et les alertes sur les activités inhabituelles ne bloquent rien, mais ils raccourcissent le temps pendant lequel un événement passe inaperçu.
3. **Les mesures correctives agissent après coup** pour limiter les dégâts et rétablir le fonctionnement normal. Les sauvegardes et les procédures de restauration, les plans de réponse aux incidents, l'application de correctifs et les systèmes de bascule relèvent de cette catégorie.

Une même technologie peut remplir plusieurs rôles. Un système de sauvegarde est correctif lorsque vous en restaurez le contenu, tandis que la surveillance qui confirme que chaque sauvegarde s'est réellement exécutée est détective.

## Qui ou quoi l'applique

Une seconde classification ignore le moment et s'intéresse au type de mécanisme qui porte la mesure. Croiser les deux points de vue donne une grille simple qui aide à repérer les cases vides d'un programme de sécurité :

| | Préventive | Détective | Corrective |
|---|---|---|---|
| **Technique (logique)**, appliquée par les systèmes | Authentification multifacteur, chiffrement | Détection d'intrusion, surveillance des journaux | Bascule automatique, restauration depuis une sauvegarde |
| **Administrative**, politiques, procédures et formations | Charte d'utilisation acceptable, sensibilisation | Revues périodiques des accès, audits | Plan de réponse aux incidents |
| **Physique**, protection du monde tangible | Serrures, lecteurs de badges | Caméras de surveillance | Matériel de rechange, site de secours |

La plupart des programmes ont besoin des trois lignes, car une politique sans application technique est facile à ignorer, et une porte verrouillée ne fait rien contre un mot de passe volé.

## Un ordinateur portable volé, plusieurs couches

Prenons le cas d'un employé dont l'ordinateur portable est dérobé dans sa voiture. Aucune mesure ne décide seule de l'issue ; ce sont les couches qui le font.

- **Le chiffrement intégral du disque** (préventif, technique) rend les fichiers illisibles pour le voleur.
- **L'authentification multifacteur et des durées de session courtes** (préventives, techniques) : la MFA empêche le voleur d'ouvrir une nouvelle connexion avec un mot de passe trouvé sur l'appareil, tandis que les sessions déjà ouvertes dans le navigateur ne restent utilisables que jusqu'à leur expiration ou leur révocation.
- **Une procédure de signalement claire** (administrative) permet de déclarer la perte le jour même.
- **La surveillance des connexions** (détective, technique) signale toute tentative d'utiliser les identifiants de l'appareil depuis un lieu inhabituel.
- **L'effacement à distance et la révocation des sessions** (correctifs, techniques) coupent l'appareil, et **les sauvegardes synchronisées** (correctives) permettent à l'employé de reprendre son travail sur une machine de remplacement.

Si le chiffrement avait fait défaut, la détection et la révocation limiteraient tout de même les dégâts. Ce chevauchement est l'idée qui sous-tend la [défense en profondeur](../principles/defense-in-depth.md) : une faille dans une couche est couverte par une autre.

## Décider quelles mesures méritent leur place

Le risque ne tombe jamais à zéro, aussi l'objectif est-il un niveau de risque acceptable à un coût raisonnable, et non une protection parfaite. Choisir des mesures, c'est mettre en balance la probabilité et la gravité de chaque menace avec ce que coûte chaque protection à construire, à exploiter et à supporter au quotidien. Les mesures sont en grande partie choisies pour combler les [faiblesses que visent les exploits](vulnerabilities-and-exploits.md) et pour protéger la confidentialité, l'intégrité et la disponibilité décrites par la [triade CIA](cia-triad.md).

Trois croyances égarent régulièrement les programmes de sécurité :

- **"Plus de mesures signifie toujours plus de sécurité."** Des mesures redondantes ou mal choisies ajoutent du coût et de la complexité sans réduire de risque significatif, et elles peuvent pousser des utilisateurs excédés à les contourner.
- **"La prévention suffit."** La prévention finit toujours par échouer, et ce sont la détection et la récupération qui empêchent un échec de tourner au désastre.
- **"Une fois déployée, une mesure continue de fonctionner."** Les mesures se dégradent à mesure que les systèmes, les menaces et le personnel évoluent. Elles doivent être testées, revues et mises à jour pour rester efficaces, c'est pourquoi le NIST accompagne son catalogue de mesures SP 800-53 d'un guide complémentaire, SP 800-53A, consacré à l'évaluation de chaque mesure.

## Références

- [NIST Glossary: Security Control](https://csrc.nist.gov/glossary/term/security_control)
- [NIST Glossary: Countermeasures](https://csrc.nist.gov/glossary/term/countermeasures)
- [NIST SP 800-53 Rev. 5, Security and Privacy Controls for Information Systems and Organizations](https://csrc.nist.gov/pubs/sp/800/53/r5/upd1/final)
- [NIST Cybersecurity Framework](https://www.nist.gov/cyberframework)
- [ISO/IEC 27002, Information Security Controls](https://www.iso.org/standard/75652.html)
- [CISA: Cyber Essentials](https://www.cisa.gov/resources-tools/resources/cyber-essentials)
