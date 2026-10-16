---
title: "Chapeaux blancs, noirs et gris : intention et autorisation, pas compétence"
description: "Un guide des étiquettes à code couleur qui classent les hackers selon qu'ils ont la permission d'agir et ce qu'ils ont l'intention de faire, plutôt que selon leur niveau de compétence."
keywords:
  - chapeau blanc
  - chapeau noir
  - chapeau gris
  - hacking éthique
  - autorisation et intention
  - classification des hackers
---

# Chapeaux blancs, noirs et gris

**Les chapeaux blancs, noirs et gris** sont des étiquettes qui classent les personnes qui sondent les systèmes informatiques ou s'y introduisent selon deux critères : ont-elles la permission d'y être, et qu'ont-elles l'intention de faire une fois à l'intérieur ? Ils ne disent rien du talent. Un intrus criminel et un testeur de sécurité rémunéré peuvent employer exactement les mêmes outils ; le chapeau indique de quel côté d'une ligne chacun se tient.

## Du western au jargon de la sécurité

La métaphore des couleurs est plus ancienne que l'informatique. Dans les premiers westerns américains, le héros portait souvent un chapeau blanc et le méchant un chapeau noir, un raccourci visuel permettant au public de les distinguer d'un coup d'œil. La culture de la sécurité a emprunté cette image parce qu'elle captait ce qui comptait : la posture morale, pas la capacité. Les étiquettes se sont attachées à l'identité du hacker à mesure que cette identité se répandait, une histoire retracée dans [Origines](origins.md) et à travers les branches décrites dans [Sous-cultures et valeurs](subcultures-and-values.md).

## La ligne de partage est l'autorisation, pas la compétence

Parce qu'un chapeau blanc et un chapeau noir peuvent recourir à des techniques identiques, la compétence ne peut pas les départager. Deux questions le font. D'abord, l'autorisation : ont-ils été invités, dans le cadre d'un contrat, du périmètre d'un programme de bug bounty, ou d'une permission explicite ? Ensuite, l'intention : voulaient-ils renforcer le système ou l'exploiter ? Le chapeau gris occupe l'espace inconfortable où l'une des réponses manque.

| Chapeau | Autorisation | Intention | Rôle typique |
|---|---|---|---|
| Blanc | Oui, dans un périmètre convenu | Améliorer la sécurité du système | Testeur d'intrusion, chercheur en bug bounty |
| Noir | Aucune | Gain personnel, perturbation ou préjudice | Intrus criminel |
| Gris | Aucune, ou ambiguë | Souvent pour exposer une faille, pas pour en profiter | Chercheur non invité qui signale ce qu'il trouve |

Le chapeau gris se définit par la contradiction : une bonne intention associée à une absence de permission. C'est ce mélange qui fait que la catégorie est débattue plutôt que célébrée.

## Une faille, trois réactions

Imaginons que quelqu'un remarque que le portail patient d'un hôpital laisse fuiter des dossiers par une adresse web prévisible. La même découverte se ramifie de trois façons :

1. **Chapeau blanc.** La personne avait déjà été mandatée pour tester le portail, ou soumet sa découverte via un programme de bug bounty ouvert. Elle documente la faille, la signale en privé et laisse l'hôpital la corriger avant que le moindre détail ne soit rendu public.
2. **Chapeau noir.** La personne récolte les dossiers pour les revendre ou pour faire chanter l'hôpital. L'acte n'est pas autorisé et l'intention est le préjudice.
3. **Chapeau gris.** La personne n'a jamais été invitée mais ne veut causer aucun tort. Elle confirme la faille et envoie un courriel à l'hôpital, ou publie un avertissement expurgé. L'intention penche vers l'éthique, pourtant accéder aux dossiers sans permission peut tout de même enfreindre la loi.

Seul le premier chemin garde les deux réponses nettes. Le troisième montre comment de bonnes intentions n'effacent pas le problème de l'autorisation, une tension examinée dans [autorisation et consentement](../ethics-and-philosophy/authorization-and-consent.md).

## Là où les couleurs se brouillent

Les chapeaux se lisent mieux comme des descriptions d'actes que comme des identités permanentes. Un testeur professionnel qui reste strictement dans son périmètre au travail pourrait, sur son temps libre, sonder un site sans invitation et glisser vers le gris. Des termes comme "red team" et "blue team" décrivent des rôles coopérant au sein d'un exercice autorisé et se situent entièrement en dehors de cette échelle.

Les étiquettes portent aussi le vieux débat du domaine sur la question de savoir si la curiosité non autorisée est jamais justifiée, le même défi exprimé dans le [Manifeste du hacker](hacker-manifesto.md). Pour celui qui apprend, l'enseignement utile est simple : ne jugez pas un hacker à la sophistication de ce qu'il a fait, mais à la question de savoir s'il avait le droit de le faire et pourquoi il l'a fait.

## Références

- [The Jargon File: cracker](http://www.catb.org/jargon/html/C/cracker.html)
- [Eric S. Raymond, How To Become A Hacker](http://www.catb.org/~esr/faqs/hacker-howto.html)
- [NIST SP 800-115, Technical Guide to Information Security Testing and Assessment](https://csrc.nist.gov/pubs/sp/800/115/final)
- [CISA Coordinated Vulnerability Disclosure Process](https://www.cisa.gov/coordinated-vulnerability-disclosure-process)
- [IETF RFC 1392: Internet Users' Glossary](https://www.rfc-editor.org/rfc/rfc1392)
