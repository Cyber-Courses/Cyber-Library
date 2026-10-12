---
title: "Méthodologie : comment une mission offensive est structurée"
description: "La catégorie Méthodologie couvre le déroulement d'une mission offensive plutôt que les techniques individuelles, répartie par phase : préparation, exécution opérationnelle et clôture."
keywords:
  - méthodologie de test d'intrusion
  - processus red team
  - phases de mission
  - règles d'engagement
  - reporting
---

# Méthodologie

La catégorie Méthodologie porte sur le déroulement d'une mission offensive, et non sur les techniques individuelles qu'elle emploie. Les techniques vivent dans les catégories centrées sur les cibles ([logiciel](../software/index.md), [réseau](../network/index.md), [physique](../physical/index.md), [matériel](../hardware/index.md), [données](../data/index.md)) ; la méthodologie est le tissu conjonctif qui décide quand et comment elles sont appliquées, comment une opération reste dans son périmètre et comment ses résultats deviennent quelque chose sur quoi une organisation peut agir.

## Pourquoi ce découpage

Les sous-catégories suivent le cycle de vie d'une mission, dans l'ordre, car les préoccupations de chaque phase sont distinctes :

- **Préparation** : tout ce qui précède le contact avec une cible, y compris le cadrage, les règles d'engagement, l'autorisation, la modélisation des menaces, la mise en place de l'infrastructure et la planification de la reconnaissance.
- **Opérationnel** : la mission elle-même, couvrant le déroulement de l'exécution, le savoir-faire, le rythme, la déconfliction, la capture de preuves et le maintien dans les limites convenues tout en s'adaptant à ce que la cible révèle.
- **Clôture** : la finalisation, y compris le nettoyage des artefacts et des accès, l'analyse des constats, le reporting et la passation qui transforme l'opération en résultats priorisés et actionnables.

Découper par phase correspond à la façon dont un opérateur pense réellement, en avançant le long d'une chronologie plutôt qu'en sautant d'une technique à l'autre. Une décision de cadrage et une décision de reporting sont des travaux de nature différente, avec des entrées différentes, aussi les garder dans des phases distinctes évite que la discipline pratique de conduite d'une mission ne se disperse à travers les catégories techniques. C'est la catégorie qui maintient le travail offensif autorisé, reproductible et utile.

## Références

- [PTES: Penetration Testing Execution Standard](http://www.pentest-standard.org/)
- [OWASP WSTG: Testing framework](https://owasp.org/www-project-web-security-testing-guide/latest/3-The_OWASP_Testing_Framework/)
