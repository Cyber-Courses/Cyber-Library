---
title: "Logiciel : techniques offensives contre les applications et les systèmes d'exploitation"
description: "La catégorie Logiciel couvre les attaques sur le code et les systèmes qui l'exécutent, répartie entre les applications qui implémentent les comportements et les systèmes d'exploitation qui les hébergent."
keywords:
  - sécurité logicielle
  - sécurité applicative
  - attaques sur les systèmes d'exploitation
  - élévation de privilèges
  - exploitation
---

# Logiciel

La catégorie Logiciel couvre le travail offensif contre le code et les systèmes qui l'exécutent, la partie la plus vaste et la plus activement développée de la bibliothèque. Elle recouvre les défauts des programmes qu'une organisation fait tourner et des systèmes d'exploitation sous-jacents, d'une erreur de logique dans un gestionnaire de requêtes web à un chemin d'élévation de privilèges dans un noyau.

## Pourquoi ce découpage

Les deux sous-catégories se divisent selon la nature du code et qui le contrôle :

- **[Applications](applications/index.md)** : les programmes qui implémentent les comportements, qu'ils s'exécutent sur l'appareil d'un utilisateur ou sur un serveur. Ces défauts vivent dans ce que les développeurs ont écrit et dans la façon dont ils ont assemblé frameworks et bibliothèques.
- **Systèmes d'exploitation** : les plateformes qui hébergent les applications, où les attaques visent le noyau, les services système, les pilotes et le modèle de privilèges et d'isolation dont tout ce qui est au-dessus dépend.

Le découpage reflète une frontière réelle, tant de propriété que de technique. Les défauts applicatifs se corrigent généralement en modifiant le code ou la configuration propre à l'application, et on les atteint par les interfaces que l'application expose. Les défauts des systèmes d'exploitation se corrigent par l'éditeur de la plateforme ou l'administrateur système, et on les atteint par les appels système, les frontières de privilèges locales et les services. Une évaluation passe typiquement de l'un à l'autre, en utilisant un point d'ancrage applicatif pour atteindre l'hôte puis le système d'exploitation pour approfondir le contrôle, mais les techniques et les personnes qui les corrigent sont distinctes, ce qui explique que la catégorie se ramifie d'abord à cet endroit.

## Références

- [OWASP Testing Guide](https://owasp.org/www-project-web-security-testing-guide/)
- [MITRE ATT&CK](https://attack.mitre.org/)
