---
title: "Listage de répertoire OGNL : confirmer l'évaluation d'expression Struts2 à l'aveugle"
description: "Une sonde d'évaluation OGNL Struts2 utilise des opérations arithmétiques et de chaîne dans %{...} pour prouver que le langage d'expression s'exécute, en marquant la réponse sans toucher au sandbox d'accès aux membres."
keywords:
  - sonde d'évaluation OGNL
  - détection OGNL Struts2
  - marqueur arithmétique OGNL
  - injection OGNL à l'aveugle
  - value stack
---

# Listage de répertoire

Avant tout payload d'exécution, la question est de savoir si le point d'injection atteint réellement l'évaluation OGNL. Un appel direct à `Runtime` est bloqué par le sandbox d'accès aux membres et peut aussi être filtré, donc une sonde de confirmation évite les deux : elle n'utilise que des opérations arithmétiques et de chaîne, que le sandbox ne restreint jamais, et guette le résultat évalué dans la réponse. C'est l'analogue OGNL d'un listage de répertoire ou d'un contrôle de marqueur, listant ce que la value stack résout plutôt qu'invoquant le runtime de l'hôte.

## Marqueur arithmétique

Le signal le plus propre est une opération arithmétique que le serveur calcule. Si le paramètre est renvoyé et qu'OGNL s'exécute, la réponse porte le produit plutôt que le littéral :

```
%{233*233}
```

Une réponse contenant `54289` prouve l'évaluation. La même idée amorce une variable nommée et la relit, ce qui survit aux contextes où un nombre nu est converti et éliminé :

```
%{#probe=233*233}
```

## Marqueur d'évaluation de chaîne

Une transformation de chaîne est plus difficile à confondre avec des nombres fortuits dans la page. La sonde de type J2EEScan construit un token connu via `replace`, de sorte que le marqueur n'apparaît que si OGNL a évalué l'appel :

```
${"zkz".replace("z","s")}
```

Une réponse contenant `sks` confirme à la fois qu'OGNL s'est exécuté et que les appels de méthode sur `String` de base sont atteignables. Une variante multiplication-plus-concaténation combine les deux signaux en un seul token improbable :

```
%{"res_"+(233*233)}
```

## Énumérer le contexte

Une fois l'évaluation confirmée, la même grammaire sûre pour le sandbox lit la stack pour lister ce qui est atteignable, ce qui oriente le payload d'escalade :

```
%{#context}
%{#context.keySet()}
%{@java.lang.System@getProperty("user.dir")}
```

`#context` énumère les clés du contexte OGNL, y compris l'entrée d'accès aux membres que la page [exécution de code à distance](remote-code-execution.md) nettoie, et `user.dir` renvoie le répertoire de travail qui cadre les chemins de fichiers ultérieurs. Atteindre `@java.lang.System@getProperty` sans erreur indique déjà un sandbox relâché ou déjà nettoyé sur cette version.

## Outils

- **Burp Suite** : Repeater pour envoyer des sondes d'évaluation arithmétiques et de remplacement de chaîne.
- **J2EEScan** : extension Burp dont la sonde OGNL utilise le marqueur "zkz".replace.

## Références

- [Apache Commons OGNL Language Guide](https://commons.apache.org/dormant/commons-ognl/language-guide.html)
- [Apache Struts2 Security](https://struts.apache.org/security/)
