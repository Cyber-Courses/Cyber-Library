---
title: "Responsabilité : relier les actions aux identités responsables"
description: "Le principe de sécurité selon lequel chaque action significative peut être reliée à une identité responsable grâce à une journalisation fiable et à la non-répudiation."
keywords:
  - responsabilité
  - journalisation d'audit
  - non-répudiation
  - traçabilité
  - attribution
  - principes de sécurité
---

# Responsabilité

**La responsabilité** est la propriété selon laquelle chaque action significative dans un système peut être reliée à l'identité qui en est responsable. Elle repose sur une identification fiable, des enregistrements dignes de confiance de ce qui s'est passé, et une intégrité suffisante de ces enregistrements pour qu'un acteur ne puisse pas nier de façon plausible ce qu'il a fait. Là où l'authentification répond à "qui êtes-vous" et l'autorisation à "que pouvez-vous faire", la responsabilité répond à "qui a fait cela, et pouvons-nous le prouver". Elle transforme un système opaque en quelque chose qui peut être examiné, expliqué et, si nécessaire, utilisé comme preuve.

## La chaîne qui relie une action à une identité responsable

La responsabilité se comprend mieux comme une chaîne. Chaque maillon doit tenir, sinon la question "qui a fait cela ?" se termine par une supposition.

1. **Une identité unique agit.** Chaque personne et chaque service dispose de son propre compte, de sorte qu'aucune action ne se cache derrière une connexion partagée. Ce maillon repose sur [l'authentification et l'autorisation](authentication-and-authorization.md) : le système doit savoir qui est l'acteur et ce qu'il était autorisé à faire.
2. **L'action est enregistrée.** La journalisation d'audit capture les événements pertinents pour la sécurité, tels que les connexions, les changements de permissions et l'accès à des enregistrements sensibles, avec un horodatage et l'identité à l'origine de l'action.
3. **L'enregistrement est protégé.** Un stockage en ajout seul, une collecte centralisée à l'écart des systèmes surveillés et une rétention contrôlée empêchent que les enregistrements soient modifiés ou supprimés après coup.
4. **Quelqu'un le lit.** Les journaux sont agrégés et analysés afin que les activités inhabituelles soient remarquées plutôt qu'enfouies.

## Une trace concrète : qui a supprimé la table des clients ?

Imaginons qu'une base de données de production perde une table de clients tard un vendredi soir. Avec un dispositif de responsabilité en place, l'enquête se déroule ainsi :

- Le journal d'audit de la base de données montre une instruction `DROP TABLE` à 22 h 14, émise par le compte `j.martin`, et non par une connexion générique `dbadmin`.
- Le journal du fournisseur d'identité montre que `j.martin` s'est connecté à 22 h 09 avec une authentification multifacteur depuis un ordinateur portable connu de l'entreprise.
- Comme les deux systèmes synchronisent leurs horloges sur la même source de temps, les deux événements s'alignent dans le bon ordre.
- Les journaux étaient transmis en temps réel vers un stockage central que les administrateurs de bases de données ne peuvent pas modifier, de sorte que l'enregistrement est bien celui écrit à 22 h 14.

L'enquête dispose désormais d'un point de départ concret : une seule personne à interroger, et des preuves difficiles à contester. Qu'une seule de ces conditions disparaisse, et la trace s'effondre. Un compte administrateur partagé signifie que n'importe qui aurait pu le faire. Des horloges non synchronisées signifient que la connexion et la suppression pourraient ne même pas sembler liées. Des journaux stockés sur le même serveur auraient pu être effacés en même temps que la table.

## Là où la chaîne se rompt

| Maillon faible | Effet sur l'attribution | Remède habituel |
|---|---|---|
| Comptes partagés ou génériques | N'importe quelle action peut appartenir à plusieurs personnes | Un compte par personne et par service ; accès négocié par un intermédiaire pour les tâches à privilèges |
| Journaux non protégés | Un attaquant ou un acteur interne peut modifier ou supprimer les preuves | Transférer les journaux hors de l'hôte vers un stockage en ajout seul et à accès contrôlé |
| Dérive d'horloge entre les systèmes | L'ordre réel des événements ne peut pas être reconstitué | Synchroniser chaque système sur une source de temps commune et fiable |
| Tout journaliser sans plan | Les signaux utiles sont noyés dans le bruit, et les coûts de stockage et de confidentialité augmentent | Décider quels événements comptent, et les journaliser de manière cohérente |
| Journaux que personne n'examine | Les abus sont enregistrés mais jamais remarqués | Agrégation centralisée avec alertes et examen périodique |

Restreindre les accès aide également : avec le [moindre privilège](least-privilege.md), moins de personnes peuvent effectuer une action sensible donnée, si bien que l'ensemble des acteurs plausibles est réduit avant même d'ouvrir les journaux.

## Non-répudiation : faire tenir l'enregistrement en cas de litige

Une entrée de journal ordinaire indique qu'un compte a fait quelque chose. La non-répudiation va plus loin et rend difficile pour l'acteur de le nier ultérieurement. Le NIST la décrit comme l'assurance que l'expéditeur d'une information reçoit une preuve de remise et que le destinataire reçoit une preuve de l'identité de l'expéditeur, de sorte qu'aucun des deux ne puisse nier par la suite avoir traité l'information.

Les signatures numériques sont l'outil habituel. Seul le détenteur d'une clé privée peut produire une signature valide, et n'importe quel tiers peut la vérifier avec la clé publique correspondante. Les horodatages cryptographiques ajoutent la preuve du moment où quelque chose a été signé. Ensemble, ils lient une action à un acteur d'une manière qui résiste à un désaccord, ce qui compte pour les contrats, les approbations financières et les publications de code. La garantie ne vaut que ce que vaut la protection de la clé privée : une clé volée permet à quelqu'un d'autre de signer au nom de son propriétaire.

## Pourquoi les organisations y investissent

Une responsabilité solide modifie les comportements avant même que quoi que ce soit ne tourne mal : les gens agissent différemment lorsqu'ils savent que leurs actions sont attribuées et enregistrées. Quand un problème survient, elle permet la détection et l'investigation plutôt qu'un jeu de devinettes. Elle sert aussi la conformité, puisque de nombreuses réglementations et normes exigent qu'une organisation puisse démontrer qui a accédé à des données sensibles et quand ; la famille de contrôles AU (Audit and Accountability) du NIST SP 800-53 est une référence courante.

La responsabilité coûte le moins cher lorsqu'elle est planifiée tôt. Décider quels événements journaliser, où ils sont envoyés et comment ils sont protégés fait partie de la [sécurité dès la conception](security-by-design.md), et les architectures fondées sur le [Zero Trust](zero-trust.md) produisent naturellement les enregistrements par requête dont dépend l'attribution.

## Références

- [NIST Glossary: Accountability](https://csrc.nist.gov/glossary/term/accountability)
- [NIST Glossary: Non-repudiation](https://csrc.nist.gov/glossary/term/non_repudiation)
- [NIST SP 800-53 Rev. 5, Security and Privacy Controls (AU Audit and Accountability)](https://csrc.nist.gov/pubs/sp/800/53/r5/upd1/final)
- [NIST SP 800-92, Guide to Computer Security Log Management](https://csrc.nist.gov/pubs/sp/800/92/final)
- [OWASP: Logging Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Logging_Cheat_Sheet.html)
- [ISO/IEC 27001 Information Security Management](https://www.iso.org/standard/27001)
