---
title: "Orchestration : attaquer le plan de contrôle du cluster de conteneurs"
order: 3
description: "Attaquer la couche d'orchestration qui planifie et gère les conteneurs sur une flotte d'hôtes. Kubernetes est la cible dominante : énumérer le cluster, atteindre les composants exposés, escalader via le RBAC, s'évader d'un pod vers son nœud, se déplacer latéralement et persister."
keywords:
  - orchestration de conteneurs
  - kubernetes
  - plan de contrôle du cluster
  - sécurité k8s
  - attaque d'orchestration
---

# Orchestration

L'orchestration planifie les conteneurs sur de nombreux hôtes et leur donne une identité, un réseau et un magasin de secrets partagés. Ce plan de contrôle est la surface d'attaque : le compromettre atteint chaque charge de travail et chaque nœud d'un seul coup. Kubernetes est la cible écrasante, de sorte que cette couche s'organise autour de lui, du premier point d'ancrage dans un pod jusqu'à la prise de contrôle du cluster et la persistance.

## Sous-thèmes

- **[Kubernetes](kubernetes/index.md)** : l'orchestrateur dominant, attaqué depuis un point d'ancrage dans un pod jusqu'au contrôle total du cluster.

## Références

- [Concepts de sécurité de Kubernetes](https://kubernetes.io/docs/concepts/security/)
- [MITRE ATT&CK : matrice Containers](https://attack.mitre.org/matrices/enterprise/containers/)
