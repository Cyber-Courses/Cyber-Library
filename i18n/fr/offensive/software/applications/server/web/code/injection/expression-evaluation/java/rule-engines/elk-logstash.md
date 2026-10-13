---
title: "Injection ELK Logstash : exécution de code via le filtre ruby"
description: "Le filtre ruby du pipeline Logstash exécute du Ruby arbitraire fourni dans son option code, donc l'injection de configuration ou d'expression dans un filtre ruby est de l'exécution Ruby et de l'exécution de commandes sur l'hôte du pipeline."
keywords:
  - Logstash
  - filtre ruby Logstash
  - injection ELK
  - injection de pipeline
  - exécution de code Ruby
---

# ELK Logstash

Logstash est l'étape d'ingestion et de transformation de la stack ELK. Un pipeline est une configuration de plugins d'entrée, de filtre et de sortie, et l'étape de filtre peut exécuter une logique arbitraire. Le filtre `ruby` est le point sensible : son option `code` est une chaîne de Ruby que Logstash compile et exécute pour chaque événement. Tout ce qui laisse une entrée non fiable atteindre une définition de pipeline, ou atteindre le `code` d'un filtre `ruby`, est de l'exécution Ruby sur l'hôte faisant tourner Logstash, qui exécute des commandes directement.

## Le filtre ruby comme voie d'exécution

Un filtre `ruby` embarque du Ruby qui s'exécute par événement :

```ruby
filter {
  ruby {
    code => "event.set('x', system('id'))"
  }
}
```

Là où un pipeline est assemblé à partir de templates, stocké dans un datastore qu'un attaquant peut écrire, ou construit à partir de champs contrôlés par l'utilisateur (un constructeur de pipeline multi-tenant, une UI qui laisse les opérateurs coller des snippets de filtre), injecter dans la chaîne `code` exécute du Ruby de l'attaquant. Ruby atteint le système d'exploitation par plusieurs équivalents, donc le sink de commande est flexible :

```ruby
system('id')          # runs and returns status
`id`                  # backticks capture stdout
IO.popen('id').read   # capture with a handle
require 'open3'; Open3.capture2('id')
```

Contrairement aux moteurs JVM, les backticks de Ruby et `system('shell string')` invoquent bel et bien un shell, donc `$(...)`, les pipes et les redirections s'étendent normalement ici :

```ruby
`id > /tmp/o 2>&1; cat /etc/passwd | head`
```

Le filtre `ruby` peut aussi charger un script autonome via son option `path`, donc là où l'attaquant peut placer ou pointer vers un fichier de script, c'est une route tout aussi directe. Capturer la sortie dans `event.set(...)` fait remonter les résultats de commande dans les documents indexés, qui peuvent alors être lus en aval.

## Les conditionnelles sont une surface plus faible

Les conditionnelles de pipeline Logstash (`if [field] == "..."`) prennent en charge une grammaire de comparaison et booléenne sur les champs d'événement mais ne sont pas un langage de scripting. Injecter dans une conditionnelle laisse un attaquant altérer la logique de routage et de filtre, abandonner ou mal router des événements, ou brancher sur des valeurs de champ façonnées, mais cela n'exécute pas de code par soi-même. Traitez les conditionnelles comme une surface de falsification de logique et le filtre `ruby` comme la voie d'exécution de code.

## Outils

- Tests manuels avec Burp Repeater ; payloads façonnés par moteur.
- **Burp Suite** : Repeater pour injecter dans les champs de pipeline ou de filtre ruby exposés via HTTP.

## Références

- [Logstash: ruby filter plugin](https://www.elastic.co/guide/en/logstash/current/plugins-filters-ruby.html)
- [Logstash: pipeline configuration](https://www.elastic.co/guide/en/logstash/current/configuration.html)
- [Logstash: conditionals](https://www.elastic.co/guide/en/logstash/current/event-dependent-configuration.html#conditionals)
