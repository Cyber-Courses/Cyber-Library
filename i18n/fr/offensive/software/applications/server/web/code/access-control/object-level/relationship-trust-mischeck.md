---
title: "Mauvais contrôle de confiance sur les relations : n'autoriser qu'un seul côté d'un graphe ou d'une jointure"
description: Graphes d'objets où l'appartenance à une organisation, une équipe ou un tenant n'est pas jointe lors de l'accès à un enregistrement enfant ou lié.
keywords:
  - cross tenant
  - appartenance à une organisation
  - accès d'équipe
  - BOLA
---

# Mauvais contrôle sur les relations

## Contexte

Les applications B2B modélisent des arêtes entre organisation, équipe, projet et rôle. Un sujet peut appartenir à l'organisation A et demander malgré tout un objet qui n'est lié qu'à l'organisation B via une table de relation. L'absence de jointures d'appartenance sur cette arête est une variante de BOLA portant sur la relation, et pas uniquement sur la table de clé primaire unique.

## Théorie

Le signal lors d'une revue de code est une requête qui vérifie `user.org_id` mais pas `object.org_id`, ou qui vérifie `project_id` sur l'enfant sans vérifier que le projet appartient bien à l'organisation. Les endpoints d'API par lot qui renvoient « tous les projets de l'utilisateur », puis prennent un identifiant de projet depuis un second appel, sont faciles à désaligner.

## Pratique

### Transfert d'identifiant inter-tenant en laboratoire

- Créez le même type d'objet dans l'organisation A et l'organisation B avec deux utilisateurs de test. Appelez la lecture ou la mise à jour de l'objet de B tout en étant authentifié comme un utilisateur qui ne devrait voir que A, en utilisant des chemins qui incluent le contexte d'organisation ou d'équipe de A dans l'URL.

## Outils

- **Burp Suite**
- **curl**
