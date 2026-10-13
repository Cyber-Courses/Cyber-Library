---
order: 4
title: "Injection XSLT"
description: "Des feuilles de style ou des paramètres de transformation influencés par l'attaquant lui permettent de lire des fichiers, d'atteindre des services internes, de récupérer des feuilles de style distantes et d'exécuter du code dans le moteur de transformation."
keywords:
  - injection XSLT
  - transformation côté serveur
  - Xalan
  - Saxon
  - libxslt
---

# XSLT

XSLT transforme une source XML en HTML, en texte ou en un autre document XML à l'aide d'une feuille de style, et les moteurs côté serveur exécutent cette transformation dans le processus : **Xalan** et **Saxon** sur la JVM, **libxslt** sous PHP et Python, et la pile **.NET** `System.Xml.Xsl`. L'injection survient lorsque toute partie de la feuille de style, ou un paramètre inséré dans celle-ci, provient d'une entrée non fiable, car une feuille de style est elle-même du code exécutable ayant accès à la résolution de documents, aux fonctions d'extension et, dans plusieurs moteurs, au runtime hôte.

La portée d'une injection XSLT dépend du processeur et de sa configuration. Chaque moteur résout les URI via `document()`, donnant lecture de fichier et SSRF. `xsl:import` et `xsl:include` intègrent des feuilles de style distantes. Les fonctions d'extension et les blocs `xsl:script` ou `msxsl:script` intégrés exécutent du Java, du JScript ou du C# directement, faisant escalader une transformation vers une exécution de code. L'identification du moteur via `system-property()` choisit la bonne charge utile.

Ce sous-arbre couvre l'abus de la fonction document, l'abus de fonctions d'extension et de scripts intégrés, et l'import de feuille de style distante.

## Pages

- **[Abus de la fonction document](document-function-abuse.md)** : la fonction XSLT document() résout des URI contrôlées par l'attaquant au moment de la transformation, donnant lecture de fichier local et server-side request forgery contre des points d'accès internes...
- **[Abus de fonctions d'extension](extension-function-abuse.md)** : les fonctions d'extension Java et .NET et les blocs xsl:script / msxsl:script intégrés exécutent du code du runtime hôte quand une feuille de style ou ses paramètres sont influencés par l'attaquant...
- **[Abus d'import de feuille de style](stylesheet-import-abuse.md)** : xsl:import et xsl:include résolvent des URI href contrôlées par l'attaquant au moment de la compilation, intégrant une feuille de style distante qui fournit templates, appels d'extension et S...

## Outils

- **Burp Suite** : injection de fragments de feuille de style et de paramètres, analyse à la recherche d'injection XSLT.
- **Burp Collaborator** : confirmation du SSRF aveugle depuis les récupérations au moment de la transformation.
- Tests manuels avec des charges utiles `document()`, de fonctions d'extension et `system-property()`.

## Références

- PortSwigger : XSLT injection
- PayloadsAllTheThings : XSLT Injection
