---
order: 8
title: "Injection de template côté serveur Rust"
description: "SSTI dans les moteurs de template Rust : les expressions de Tera analysées à l'exécution, pourquoi les moteurs à la compilation ne sont pas injectables, et le plafond de divulgation d'information qu'impose la sandbox de Tera."
keywords:
  - SSTI Rust
  - Tera
  - injection de template
  - templates à la compilation
---

# Rust

La plupart des moteurs de template Rust (Askama, Maud, Yarte) compilent les templates au moment du build à partir de fichiers de l'arborescence source : il n'y a donc pas de sink à l'exécution qui analyse un texte de template contrôlé par l'attaquant, et ils ne sont pas injectables. Le cas qui compte pour la SSTI est Tera, un moteur inspiré de Jinja2 qui analyse les templates à l'exécution et qui est donc atteignable lorsqu'une application appelle `Tera::one_off` ou rend une chaîne de template construite à partir d'une entrée utilisateur.

Tera est un autre cas honnête et limité : comme Go, il évalue des expressions et appelle des filtres et fonctions enregistrés, mais n'expose aucun chemin vers le runtime Rust, si bien que l'impact est normalement une divulgation d'information plutôt qu'une RCE.

## Moteurs

- **[Tera](tera.md)** : confirmation avec `{{7*7}}`, énumération du contexte et des filtres, et les limites qui la cantonnent à la divulgation.

## Références

- Documentation de Tera (Keats/tera)
- PortSwigger Web Security Academy: Server-side template injection
