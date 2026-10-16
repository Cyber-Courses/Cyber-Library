---
title: "Menaces et acteurs de la menace : qui attaque et pourquoi"
description: "Ce qu'est une menace au sens de la sécurité, les types d'acteurs à l'origine des menaces, et les motivations et capacités qui façonnent leur comportement."
keywords:
  - menaces
  - acteurs de la menace
  - modélisation des menaces
  - motivation des adversaires
  - capacités des attaquants
  - renseignement sur les cybermenaces
---

# Menaces et acteurs de la menace

**Une menace** est toute circonstance ou tout événement susceptible de nuire à un actif en exploitant une vulnérabilité. Un **acteur de la menace** est la personne, le groupe ou l'entité qui cause, ou a l'intention de causer, ce préjudice. Il est utile de bien distinguer ces deux notions : la menace décrit ce qui pourrait mal tourner, tandis que l'acteur décrit qui le provoquerait et pourquoi. Ensemble, elles permettent de se demander non seulement si une faiblesse existe, mais aussi si quelqu'un est motivé et capable de l'exploiter.

## Une menace a besoin d'une cible et d'un point d'entrée

Une menace n'a d'importance qu'au regard de quelque chose qui mérite d'être protégé (un [actif](assets-and-attack-surface.md)) et d'une faiblesse qui permet au préjudice de se produire (une [vulnérabilité](vulnerabilities-and-exploits.md)). Supprimez l'un ou l'autre, et la menace n'a plus de prise.

Deux exemples illustrent cet enchaînement :

1. **Une inondation et un centre de données.** Les serveurs qu'il abrite ont de la valeur, et le bâtiment peut être inondé. L'inondation est la menace, les serveurs sont l'actif, l'implantation en zone basse est la faiblesse.
2. **L'hameçonnage et le compte d'un employé.** Les gens peuvent être trompés, et un compte, une fois dérobé, donne accès à quelque chose d'utile. Le courriel d'hameçonnage est la menace, le compte et ce qu'il permet d'ouvrir sont l'actif, la confiance humaine est la faiblesse.

Ce que la menace met finalement en péril, c'est la confidentialité, l'intégrité ou la disponibilité de cet actif, telles que décrites par la [Triade CIA](cia-triad.md).

## Des menaces sans adversaire

Toutes les menaces ne proviennent pas d'un acteur hostile. La sécurité distingue les menaces **délibérées**, comme un intrus, des menaces **accidentelles ou environnementales** : une coupure de courant, une panne matérielle, une tempête ou une erreur de bonne foi d'un employé. Les recommandations du NIST en matière d'appréciation des risques les traitent comme des catégories de sources de menace distinctes des sources adverses, car elles appellent des questions différentes. Un adversaire s'adapte à vos défenses ; un disque défaillant, non. La suite de cette page porte sur le versant délibéré, celui où intervient un acteur de la menace.

## Cinq familles d'acteurs de la menace

Les acteurs de la menace sont couramment regroupés selon qui ils sont et ce qui les motive. Dans la pratique, les frontières sont floues (un État peut tolérer ou employer des criminels, un initié peut être recruté par un acteur externe), mais ces familles constituent un point de départ utile.

| Acteur | Motivation typique | Capacité typique | Manifestations fréquentes |
|---|---|---|---|
| Cybercriminels | L'argent | Moyenne à élevée, souvent achetée ou louée | Rançongiciels, fraude, vol de données revendables |
| Acteurs étatiques | Espionnage, sabotage, avantage stratégique pour un gouvernement | Élevée : financement, temps, outils sur mesure | Intrusions longues et discrètes visant l'administration, la défense et les infrastructures critiques |
| Hacktivistes | Une cause politique ou sociale | Faible à moyenne | Défacement de sites web, fuites, perturbations calées sur l'actualité |
| Initiés | Rancœur, argent ou simple négligence | Un accès légitime, qui compte souvent plus que la compétence | Détournement d'accès par des employés et prestataires actuels ou anciens |
| Amateurs de sensations et passionnés | Curiosité, réputation, le défi en lui-même | Très variable | Sondage de systèmes exposés, intrusions ponctuelles médiatisées |

Des catalogues publics comme la liste des Groups de MITRE ATT&CK recensent des acteurs nommés au sein de ces familles, ainsi que les techniques que chacun a été observé utiliser.

## Motivation, capacité, opportunité

Deux dimensions décrivent tout acteur :

- La **motivation** est la raison pour laquelle il agit : l'argent, l'information, la perturbation, l'idéologie ou l'ego.
- La **capacité** est ce qu'il peut réellement faire, en fonction de ses compétences, de son financement, de son temps et de ses outils.

Un acteur très motivé mais peu compétent pose un problème différent de celui d'un acteur patient et bien financé. Les analystes ajoutent souvent un troisième facteur, l'**opportunité** : l'accès ou l'exposition dont un acteur a besoin pour pouvoir agir.

Réduire l'un de ces trois facteurs diminue le risque, mais les défenseurs ont rarement prise sur les deux premiers. On ne peut pas rendre un criminel moins intéressé par l'argent, ni un gouvernement moins bien financé. L'opportunité, en revanche, est différente : appliquer les correctifs, réduire les services exposés et limiter les accès grâce aux [contrôles et contre-mesures](controls-and-countermeasures.md) la réduisent directement.

## Un serveur exposé, trois attaquants différents

Imaginez une petite entreprise qui exploite un serveur de fichiers non corrigé et exposé sur Internet. Trois acteurs pourraient l'atteindre, et chacun remet en cause une idée reçue.

- **Le scanner automatisé.** Les outils d'un groupe criminel balaient Internet à la recherche de cette faille précise et déposent un rançongiciel sur chaque cible trouvée. Personne n'a choisi cette entreprise ; elle était simplement accessible et non corrigée. L'idée "nous sommes trop petits pour être une cible" ne tient pas, car une grande partie de la criminalité est opportuniste et automatisée.
- **L'équipe organisée.** L'accès est revendu par un spécialiste à un autre qui se charge de l'extorsion, tandis qu'un troisième négocie le paiement. La plupart des acteurs de la menace ne sont pas des génies solitaires, mais des groupes organisés avec des rôles spécialisés, des outils partagés et même des services d'assistance.
- **L'administrateur sur le départ.** Un prestataire dont le compte n'a jamais été désactivé copie le contenu du serveur avant de partir. Les initiés, et les simples accidents, représentent une large part des incidents réels : se concentrer uniquement sur les acteurs externes laisse donc une lacune.

Une même faiblesse, trois acteurs très différents : comprendre qui pourrait exploiter une vulnérabilité, et pourquoi, c'est ce qui transforme une liste de failles en une vision des risques hiérarchisée.

## Références

- [NIST Glossary: Threat](https://csrc.nist.gov/glossary/term/threat)
- [NIST Glossary: Threat Actor](https://csrc.nist.gov/glossary/term/threat_actor)
- [NIST SP 800-30 Rev. 1, Guide for Conducting Risk Assessments](https://csrc.nist.gov/pubs/sp/800/30/r1/final)
- [MITRE ATT&CK: Groups](https://attack.mitre.org/groups/)
- [CISA: Cyber Threats and Advisories](https://www.cisa.gov/topics/cyber-threats-and-advisories)
- [ENISA: Threats and Trends](https://www.enisa.europa.eu/topics/cyber-threats/threats-and-trends)
