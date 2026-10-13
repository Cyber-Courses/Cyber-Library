---
order: 3
title: "Contrôle d'accès dans les applications web : endpoints, objets, propriétés et autorisation défaillante (BOLA/IDOR)"
description: "Contrôle d'accès défaillant dans le code web côté serveur : portée des endpoints, des objets et des propriétés, plus la confiance envers les en-têtes et les systèmes amont."
keywords:
  - contrôle d'accès
  - autorisation
  - BOLA
  - IDOR
  - contrôle d'accès défaillant
---

# Contrôle d'accès

Le **contrôle d'accès** (autorisation) répond à la question de savoir si un appelant est autorisé à effectuer une action sur des données. Les défaillances sont fréquentes et à fort impact. Organisez les tests et la documentation selon l'endroit **où** le contrôle échoue :

| Sous-thème | Question |
|----------|----------|
| [Niveau endpoint](endpoint-level/index.md) | Cette route ou fonction est-elle restreinte aux bons rôles ou tenants ? |
| [Niveau objet](object-level/index.md) | Cet utilisateur peut-il accéder à **cet enregistrement précis** (ID, slugs, ressources imbriquées) ? |
| [Niveau propriété](property-level/index.md) | L'utilisateur peut-il modifier des champs qu'il ne devrait pas (mass assignment, mises à jour partielles) ? |
| [Frontière de confiance](trust-boundary/index.md) | L'application croit-elle les en-têtes, les certificats clients ou l'identité amont sans liaison solide ? |
