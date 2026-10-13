---
title: "Injection dans text/template et html/template de Go"
description: "Exploiter l'injection de template Go : confirmation par le comportement de {{7*7}}, énumération des champs et appel de méthodes sur l'objet de données, lecture du modèle de données, et les limites qui empêchent généralement la RCE."
keywords:
  - injection de template Go
  - text/template
  - html/template
  - built-in call
  - invocation de méthode
---

# Templates

Les templates Go utilisent `{{ }}`, mais `{{7*7}}` n'affiche pas `49` : l'arithmétique n'est pas une syntaxe d'expression dans les templates Go, et `{{7*7}}` produit une erreur d'analyse. Les sondes fiables sont `{{.}}`, qui affiche l'intégralité de l'objet de données passé au template, et des appels du style `{{printf "%d" (len "aaa")}}` utilisant les fonctions built-in. Une erreur d'analyse mentionnant `template:` sur une entrée malformée permet aussi d'identifier le moteur.

Ce que le template peut faire est borné par conception. Il peut :

- Afficher l'objet de données et parcourir ses champs exportés : `{{.}}`, `{{.SomeField}}`, `{{.User.Email}}`.
- Appeler des méthodes exportées sur les données (et sur leurs résultats), avec des arguments : `{{.SomeMethod "arg"}}`.
- Utiliser les fonctions built-in (`print`, `printf`, `call`, `index`, `len`, `slice`, et ainsi de suite).

Il ne peut pas importer de paquets, instancier des types, ni atteindre `os`/`exec` de lui-même. Le premier impact est donc la divulgation d'information : dumper `{{.}}` fuite fréquemment l'ensemble du struct de contexte, qui peut contenir des secrets, des jetons, de la configuration interne ou des données de requête que le développeur n'avait pas l'intention d'exposer.

L'escalade vers l'exécution de code dépend entièrement des méthodes accessibles depuis l'objet de données. Si une méthode exportée dans le graphe d'objets accessible exécute une commande, écrit un fichier ou réalise une autre action sensible, le template peut l'appeler. Le built-in `call` invoque une valeur de fonction passée dans les données, de sorte qu'un contexte qui expose un champ de type fonction élargit ce qui est accessible :

```gotemplate
{{ call .SomeFunc "id" }}
{{ .Cmd.Run }}
```

Le workflow pratique est : confirmer avec `{{.}}`, énumérer les champs et méthodes exportés, lire tout ce qui est sensible dans le contexte dumpé, puis chercher une méthode accessible ayant un effet de bord. Traitez la SSTI Go comme une divulgation d'information par défaut et n'escaladez que lorsque les propres types de l'application fournissent au template une méthode dangereuse. `html/template` ajoute un échappement de sortie contextuel (de sorte que les valeurs reflétées sont encodées en HTML) mais ne change pas ce que le langage de template peut atteindre, de sorte que l'énumération et l'analyse des appels de méthode sont identiques.

## Outils

- Énumération manuelle ; tplmap a un support Go limité

## Références

- Documentation Go : text/template (actions, functions), html/template
- PortSwigger Web Security Academy: Server-side template injection
