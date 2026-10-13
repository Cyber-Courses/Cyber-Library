---
title: "Injection OGNL : abus de l'Object-Graph Navigation Language dans Struts2"
description: "Apache Struts2 évalue l'OGNL issu des données de requête via %{...} et ${...}, et une fois le sandbox d'accès aux membres nettoyé, l'expression atteint Runtime et ProcessBuilder pour l'exécution de code."
keywords:
  - injection OGNL
  - Struts2 OGNL
  - "expression %{}"
  - sandbox d'accès aux membres
  - RCE OGNL
---

# OGNL

OGNL (Object-Graph Navigation Language) est le langage d'expression que Struts2 utilise pour déplacer des valeurs entre la requête, la value stack et la vue. Les attributs de tag, la conversion de type et les noms de paramètres sont tous évalués comme de l'OGNL, écrits `%{...}` dans les tags et atteignables sous `${...}` dans certains contextes. Lorsque des données de requête transitent vers une chaîne que Struts évalue ensuite, via une double évaluation forcée, un nom de paramètre conçu à dessein ou un attribut de tag construit à partir d'une entrée, la grammaire OGNL s'exécute face à la value stack.

OGNL est un langage objet complet : il nomme des classes, construit des objets et appelle des méthodes. Struts2 moderne enveloppe l'évaluation dans un sandbox, un `SecurityMemberAccess` avec des classes et des noms de package exclus appliqués au travers du contexte OGNL, donc un appel nu à `java.lang.Runtime` est bloqué. Les pages d'exploitation ci-dessous couvrent à la fois la preuve de l'évaluation sans toucher au sandbox et le nettoyage de la restriction d'accès aux membres pour atteindre le runtime.

- **[Listage de répertoire](directory-listing.md)** : la sonde d'évaluation arithmétique et de chaîne à l'aveugle qui confirme l'évaluation OGNL avant tout appel d'exécution.
- **[Exécution de code à distance](remote-code-execution.md)** : restaurer l'accès aux membres et atteindre `Runtime` et `ProcessBuilder`.
- **[Inclusion de fichier distant](remote-file-inclusion.md)** : construire `java.io.File` et `java.net.URL` pour lire des fichiers locaux et récupérer du contenu distant.

## Outils

- **Burp Suite** : Repeater et Intruder pour l'injection OGNL dans les puits Struts2.
- **J2EEScan** : extension Burp qui détecte l'injection d'expression OGNL et Struts.
- **struts-pwn** : outil de PoC pour les vecteurs OGNL de Struts2.

## Références

- [Apache Commons OGNL Language Guide](https://commons.apache.org/dormant/commons-ognl/language-guide.html)
- [Apache Struts2 Security](https://struts.apache.org/security/)
- [PayloadsAllTheThings: Java OGNL Injection](https://github.com/swisskyrepo/PayloadsAllTheThings)
