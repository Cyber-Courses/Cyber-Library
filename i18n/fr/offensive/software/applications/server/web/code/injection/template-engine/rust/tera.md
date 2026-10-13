---
title: "Injection de template côté serveur Tera en Rust"
description: "Exploitation d'une SSTI Tera en Rust : confirmation avec {{7*7}}, énumération du contexte de rendu et des filtres intégrés pour la divulgation d'information, et le sink Tera::one_off et ses limites."
keywords:
  - SSTI Tera
  - Tera one_off
  - divulgation du contexte
  - injection de template Rust
---

# Tera

Tera analyse les templates à l'exécution et utilise des expressions `{{ }}` et des instructions `{% %}` de style Jinja2. Confirmez avec `{{7*7}}` qui rend `49`. Contrairement à Jinja2, Tera n'a pas de graphe d'objets atteignant l'interpréteur : les valeurs proviennent uniquement du contexte de rendu fourni par l'application, et le comportement appelable se limite aux filtres, tests et fonctions enregistrés. Il n'y a pas de parcours d'attributs vers un module, pas de `__globals__`, et aucune construction qui importe ou lance un processus.

L'impact réaliste est la divulgation d'information. Videz ce que le contexte expose et énumérez les filtres intégrés, qui peuvent transformer et faire fuiter des données :

```tera
{{ __tera_context }}
{{ some_var }}
{{ get_env(name="PATH") }}
```

`__tera_context` affiche le contexte de rendu sérialisé, révélant souvent des variables que la page n'avait pas l'intention de montrer. La fonction intégrée `get_env` lit les variables d'environnement lorsqu'elle n'a pas été retirée, ce qui peut faire fuiter des secrets et de la configuration. Les fonctions personnalisées enregistrées sont le seul chemin d'escalade : si l'application a enregistré une fonction Tera qui effectue une action sensible (exécute une commande, lit un fichier), le template peut l'appeler par son nom avec des arguments. Énumérez les fonctions enregistrées et testez chacune d'elles.

Le sink est `Tera::one_off(user_input, &context, autoescape)` ou le rendu d'une chaîne de template assemblée à partir d'une entrée utilisateur ; les templates chargés depuis les fichiers du projet lui-même ne sont pas contrôlés par l'attaquant. Considérez Tera comme limité à la divulgation par défaut, fouillez `__tera_context` et `get_env` à la recherche de secrets, et n'escaladez qu'au travers d'une fonction enregistrée par l'application ayant un effet de bord.

## Outils

- Énumération manuelle

## Références

- Documentation de Tera : filtres et fonctions intégrés, one_off
- PortSwigger Web Security Academy: Server-side template injection
