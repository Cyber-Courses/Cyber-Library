---
title: "Contrôle d'accès au niveau objet : IDOR, BOLA et autorisation par enregistrement dans les API"
description: "Références d'objets directes non sécurisées et problèmes connexes : accès non autorisé à des données identifiées par des ID, des slugs ou des relations entre dossiers."
keywords:
  - autorisation au niveau objet
  - BOLA
  - IDOR
  - référence d'objet directe non sécurisée
---

# Niveau objet

Le contrôle d'accès au **niveau objet** (BOLA / IDOR) est l'incapacité à vérifier que le sujet authentifié peut accéder à **cette** ressource précise. Le serveur accepte une référence (ID, UUID, slug) mais n'impose pas la propriété, l'appartenance au tenant ou le rôle sur **cette** instance.

## Pages

- [Manipulation de référence directe (IDOR)](direct-reference-tampering.md)
- [Opération d'objets par lot](batch-object-operation.md)
- [Recherche par clé alternative](alternate-key-lookup.md)
- [Objet imbriqué sans contrôle de propriété](nested-object-without-ownership-check.md)
- [ID encodés prévisibles](predictable-encoded-ids.md)
- [Devinette d'identifiant public](public-identifier-guessing.md)
- [Mauvais contrôle de confiance de relation](relationship-trust-mischeck.md)
