---
order: 1
title: "Injection de template côté serveur en Go"
description: "SSTI dans text/template et html/template de Go : ce que le moteur expose et n'expose pas, les appels de méthode sur les données injectées, et pourquoi l'exécution de commande n'est généralement pas accessible depuis le template seul."
keywords:
  - SSTI Go
  - text/template
  - html/template
  - injection par appel de méthode
---

# Go

Le templating standard de Go est `text/template` et son frère auto-échappant `html/template`. Les deux sont délibérément restreints : un template peut référencer des champs et appeler des méthodes sur les données qui lui sont passées, mais il ne peut pas importer de paquets, construire des types arbitraires, ni atteindre le runtime. Cela fait de Go un cas honnête important : injecter de la syntaxe de template produit rarement l'exécution de commande comme le font Jinja2 ou FreeMarker.

## Moteurs

- **[text/template et html/template](go-templates.md)** : détection, énumération des champs et méthodes, divulgation d'information, et les conditions d'appel de méthode sous lesquelles l'impact escalade.

## Références

- Documentation Go : text/template, html/template
- PortSwigger Web Security Academy: Server-side template injection
