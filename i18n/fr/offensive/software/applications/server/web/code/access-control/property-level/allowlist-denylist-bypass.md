---
title: "Contournement de liste d'autorisation et de liste de blocage sur les sérialiseurs, les binders et les garde-fous d'attributs d'ORM"
description: Évasion de filtre sur les noms de champs liés via des variantes de casse, des homoglyphes Unicode, des clés dupliquées ou des chemins imbriqués que le binder mappe quand même.
keywords:
  - liste d'autorisation de champs
  - pollution de paramètres
  - clé imbriquée
---

# Contournement de liste d'autorisation

## Contexte

Un serveur naïf retire `admin` de l'objet de premier niveau mais pas `Admin`, un homoglyphe, ou `profile.admin`. Les analyseurs qui appliquent le « dernier gagnant » sur les clés dupliquées ou qui fusionnent des objets imbriqués peuvent malgré tout définir un champ protégé. C'est un problème d'interaction entre binder et analyseur, pas une nouvelle primitive BOLA sur l'identifiant.

## Théorie

Les règles de clés dupliquées en JSON, les clés répétées en `application/x-www-form-urlencoded`, et `application/json` avec des astuces sur le `Content-Type` peuvent changer quelle valeur le framework lit. Chaque stack documente ses propres règles ; le travail offensif consiste à essayer la même clé logique dans chaque encodage que la stack accepte.

## Pratique

### Renvoyer le même champ logique avec des variantes de casse et d'imbrication

- En laboratoire, faites tourner `role`, `Role`, `profile[role]` et le `{"profile":{"role":"admin"}}` imbriqué contre la même route de mise à jour et comparez l'objet stocké.

## Outils

- **Burp Suite**
- **curl**
