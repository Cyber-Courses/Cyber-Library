---
title: "Contrôle d'accès au niveau des propriétés : mass assignment, overposting et champs sensibles modifiables"
description: "Modification non autorisée de champs d'objets via les API : mass assignment, overposting et contournements de filtres sur les colonnes modifiables."
keywords:
  - autorisation au niveau des propriétés
  - mass assignment
  - overposting
  - propriété BOLA
---

# Niveau des propriétés

Les problèmes **au niveau des propriétés** surviennent lorsqu'un utilisateur peut modifier des **champs** qu'il ne devrait pas : `role`, `ownerId`, `price`, `isAdmin`, ou des indicateurs de statut internes. La route peut être autorisée pour l'utilisateur, et l'ID d'objet peut même être le sien, mais la mise à jour doit malgré tout **restreindre** quelles propriétés sont légales pour ce rôle et cette opération.

## Pages

- [Mass assignment](mass-assignment.md)
- [Contournement de liste d'autorisation et de liste de blocage](allowlist-denylist-bypass.md)
- [Injection de propriété imbriquée](nested-property-injection.md)
- [Écrasement par mise à jour partielle](partial-update-overwrite.md)
- [Pollution de propriété](property-pollution.md)
- [Confusion de type de propriété](property-type-confusion.md)
- [Contournement d'un champ en lecture seule](readonly-field-override.md)
- [Champs de rôle, de propriétaire et de facturation modifiables](writable-sensitive-fields.md)
