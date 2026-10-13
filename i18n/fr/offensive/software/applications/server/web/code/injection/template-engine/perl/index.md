---
order: 5
title: "Injection de template côté serveur en Perl"
description: "SSTI dans le Template Toolkit de Perl : les blocs PERL et RAWPERL conditionnés par EVAL_PERL, et les plugins system/exec qui atteignent l'exécution de commande."
keywords:
  - SSTI Perl
  - Template Toolkit
  - EVAL_PERL
  - bloc PERL
---

# Perl

Le moteur dominant de Perl est Template Toolkit (TT2), qui utilise les directives `[% ... %]`. TT2 a été conçu pour séparer la logique de la présentation, de sorte que l'exécution de Perl brut est conditionnée par une option explicite plutôt que disponible par défaut. L'exploitation se scinde donc en deux cas : les blocs `PERL`/`RAWPERL` lorsque `EVAL_PERL` est activé, et le mécanisme de plugins et de filtres (notamment les directives `system` et `exec` et les fonctionnalités `Datafile`/`redirect`) qui peut atteindre l'OS ou le système de fichiers selon la configuration.

Confirmez avec `[% 7*7 %]` affichant `49`.

## Moteurs

- **[Template Toolkit](template-toolkit.md)** : les blocs `[% PERL %]` sous EVAL_PERL, les filtres `system`/`exec`, et l'écriture de fichier via `redirect`.

## Références

- Documentation Template Toolkit (template2)
- PortSwigger Web Security Academy: Server-side template injection
