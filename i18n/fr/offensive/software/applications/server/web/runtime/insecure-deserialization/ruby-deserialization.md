---
order: 4
title: "Désérialisation Ruby : chaînes de gadgets universelles Marshal et YAML"
description: "Exploiter la désérialisation Ruby : Marshal.load et YAML.load/Psych sur une entrée non fiable, et les chaînes de gadgets universelles qui atteignent l'exécution de code sans classes propres à l'application."
keywords:
  - désérialisation ruby
  - Marshal.load
  - YAML.load
  - Psych
  - chaîne de gadgets universelle
---

# Désérialisation Ruby

`Marshal.load` de Ruby et, historiquement, `YAML.load` (Psych) reconstruisent des objets Ruby arbitraires à partir d'une entrée non fiable. La reconstruction exécute des callbacks comme `init_with` et `marshal_load`, et l'écosystème Ruby dispose de **chaînes de gadgets universelles** bien connues, construites à partir de la bibliothèque standard et des dépendances Rails/Gem, de sorte que l'exploitation n'a souvent pas besoin de classes propres à l'application.

## Les sinks

- **`Marshal.load(data)`** sur des octets de l'attaquant. Marshal est un format binaire commençant par `\x04\x08` ; il apparaît dans les caches, les cookies et les stores de session (Rails a un temps mis par défaut les sessions cookie sur Marshal). Marshal est explicitement non sécurisé pour les données non fiables.
- **`YAML.load(data)`** (Psych plus ancien) construit des objets arbitraires via des tags comme `!ruby/object:` et `!ruby/hash:`. `YAML.safe_load` (et les valeurs par défaut modernes) restreignent cela ; testez ce qui est utilisé.
- **`Oj.load`** en mode compat/object, et d'autres loaders JSON/MessagePack en mode object, reconstruisent les objets Ruby de la même façon.

## Chaînes de gadgets universelles

Parce que la chaîne peut être assemblée à partir de gems présents dans presque toutes les applications, des payloads génériques existent. Un gadget Psych/YAML classique utilise `Gem::Requirement` avec une séquence `Gem::Installer`/`Gem::Package` embarquée qui atteint `Kernel.system`, et des chaînes Marshal équivalentes ciblent `ActiveSupport`/`ERB` pour évaluer le code de l'attaquant. Le flux pratique consiste à reconnaître le sink, puis à appliquer une chaîne universelle connue pour la version Ruby/Rails/psych en jeu plutôt que de fabriquer des gadgets à la main.

Une forme de payload YAML contre un loader vulnérable :

```yaml
--- !ruby/object:Gem::Requirement
requirements:
  !ruby/object:Gem::Package::TarReader
  io: &1 !ruby/object:Net::BufferedIO
  ...
```

(Les chaînes complètes sont propres à la version ; utilisez une chaîne maintenue pour les versions de gem de la cible.)

## Notes d'exploitation

- Identifiez le sink : Marshal (`\x04\x08` binaire) versus YAML (texte avec des tags `!ruby/...`) détermine la famille de payload.
- Applications Rails : vérifiez le sérialiseur de session/cookie et tout `Marshal.load` sur les entrées de cache ; un cookie signé par la clé secrète bloque l'altération, cherchez donc plutôt des caches et files non signés.
- Comme pour les autres moteurs, un chargement réussi s'exécute avant que l'application n'inspecte l'objet, donc la validation après chargement n'aide pas le défenseur ni ne vous gêne.

## Outils

- Payloads de gadgets universels maintenus pour Marshal et Psych/YAML (correspondant à la version).

## Références

- Ruby docs: Marshal (security note), Psych/YAML
- PortSwigger Web Security Academy: Insecure deserialization
