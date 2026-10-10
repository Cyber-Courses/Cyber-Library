---
title: "Ingénierie de détection : construire et maintenir les détections"
order: 6
description: "Comment l'ingénierie de détection conçoit, teste et maintient les règles et les analytiques qui sous-tendent une détection fiable, en traitant les détections comme du code."
keywords:
  - ingénierie de détection
  - détection comme code
  - règles de détection
  - test des analytiques
  - cycle de vie des règles
  - maintenance des détections
---

# Ingénierie

L'ingénierie de détection est le travail rigoureux de conception, de test et de maintenance des règles et des analytiques qui rendent la détection fiable. Elle traite les détections comme des produits d'ingénierie dotés d'un cycle de vie, plutôt que comme des recherches ponctuelles, en appliquant les pratiques du logiciel afin que la couverture soit délibérée, reproductible et durable.

Au sein de la phase de détection, l'ingénierie est ce qui maintient l'analyse efficace dans le temps. Les environnements changent, les techniques des adversaires évoluent et les règles négligées se dégradent en bruit ou en silence. L'ingénierie de détection gère cette dérive en versionnant les détections, en les validant face à une activité réaliste et en retirant celles qui ne justifient plus leur place.

En pratique, cette discipline est souvent décrite comme de la détection comme code : les règles vivent dans un contrôle de version, changent via une revue et sont testées dans des pipelines avant déploiement. Les ingénieurs font correspondre les détections à des techniques connues, les écrivent pour qu'elles soient précises et explicables, et mesurent leur comportement en faux positifs et vrais positifs. Ils documentent l'intention et les dépendances de données afin qu'une détection puisse être comprise et maintenue par d'autres. Le résultat est un catalogue géré de détections dont on peut raisonner la qualité et la couverture, plutôt qu'un amas opaque de règles que personne n'ose toucher.

## Références

- MITRE ATT&CK, Detections and Analytics
- SANS, ressources sur la Detection Engineering
