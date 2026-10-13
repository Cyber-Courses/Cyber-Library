---
order: 3
title: "Extraction XPath basée sur les erreurs"
description: "Forcer un échec de cast ou de type dans un processeur XPath 2.0 intègre le texte d'un nœud dans le message d'erreur, transformant un moteur verbeux en un canal d'extraction direct et non aveugle."
keywords:
  - XPath basé sur les erreurs
  - message d'erreur XPath
  - échec de cast
  - extraction XPath 2.0
  - fuite d'erreur XML
---

# Basé sur les erreurs

Lorsqu'un processeur XPath renvoie ses messages d'erreur au client, un échec délibérément forcé peut faire sortir le texte d'un nœud à l'intérieur de la chaîne d'erreur. La forme fiable de cette technique est spécifique aux moteurs XPath 2.0 et XQuery (Saxon, BaseX, eXist-db) : leurs constructeurs de type XSD valident leur argument et nomment la valeur fautive lorsqu'elle échoue. Les moteurs XPath 1.0 tels que libxml2, MSXML et le moteur natif .NET `System.Xml.XPath` n'ont pas de constructeurs de type XSD et ne lèvent pas d'erreur sur les mauvaises conversions, donc contre ceux-là le canal d'erreur confirme et identifie l'injection mais n'extrait pas de valeurs ; la technique ci-dessous cible donc un processeur capable de 2.0, identifié au préalable par les charges utiles d'identification en fin de page.

## Pourquoi un cast fuite des données

Un cast ou un constructeur de type comme `xs:integer(...)` analyse son argument et, en cas d'échec, rapporte ce qu'il n'a pas pu convertir. Fournir le texte d'un nœud à un constructeur numérique là où le texte n'est pas un nombre valide lève une erreur dynamique dont le message cite la valeur, par exemple `FORG0001: Cannot convert string "s3cr3t" to xs:integer`. Le mot de passe est désormais dans l'erreur.

## Forcer un échec de cast

Injectez de sorte que l'expression environnante reste valide jusqu'au constructeur, puis poussez le nœud cible dedans :

```
' or xs:integer((//user[1]/pass)) or '
```

Quand le mot de passe est non numérique, le moteur avorte avec un message de style `FORG0001` contenant la valeur. Gardez chaque fragment petit et sans ambiguïté avec `substring` :

```
' or xs:integer(substring((//user[1]/pass),1,20)) or '
```

Avancez l'offset pour lire au-delà des vingt premiers caractères, et changez le chemin pour vous déplacer entre les nœuds :

```
' or xs:integer((//user[2]/@role)) or '
```

Là où une valeur se trouve être numérique, castez-la plutôt vers un type qu'elle ne peut satisfaire, tel que `xs:date(...)` ou `xs:QName(...)`, pour que la conversion échoue quand même et rapporte le texte.

## Placer le texte directement avec fn:error

Les moteurs qui exposent `fn:error()` permettent à un attaquant de construire la chaîne de l'erreur, en y intégrant le texte d'un nœud sans dépendre d'une bizarrerie de conversion :

```
' or error(xs:QName('x'), string(//user[1]/pass)) or '
```

Le processeur fait remonter la description fournie verbatim, donc le texte du nœud apparaît directement dans le canal d'erreur.

## Fuiter la structure

Avant d'extraire des valeurs, le même mécanisme révèle la forme du document. Forcer `name()` ou `local-name()` à travers un cast qui échoue rapporte les noms d'éléments un nœud à la fois :

```
' or xs:integer(name(/*[1])) or '
```

L'erreur de conversion nomme l'élément racine ; parcourir les indices avec `name(/*[1]/*[position()=N])` cartographie l'arbre pour que l'extraction de valeurs sache où viser.

## Identifier le moteur d'abord

L'extraction de valeurs basée sur les erreurs ne fonctionne que sur un moteur 2.0, donc confirmez le processeur avant de vous engager dans des casts. Une expression délibérément mal formée renvoie une erreur de parsing spécifique au moteur :

```
'
"]
count(//
```

Un guillemet ou un crochet non équilibré produit un message dont la formulation exacte identifie le moteur (libxml2, MSXML, Saxon, .NET). Si l'empreinte est un moteur 1.0 (libxml2, MSXML ou le moteur XPath natif .NET), repliez-vous sur l'extraction booléenne aveugle via le prédicat ; seul un moteur 2.0 (Saxon, BaseX, eXist-db) expose le canal d'échec de cast ci-dessus qui lit les valeurs directement.

## Outils

- **xcat** : automatisation de l'extraction basée sur les erreurs contre les moteurs XPath 2.0.
- **Burp Repeater** : forcer des échecs de cast et lire le texte de nœud fuité.
- Tests manuels avec des charges utiles de cast `xs:integer()` et `fn:error()`.

## Références

- [OWASP: XPATH Injection](https://owasp.org/www-community/attacks/XPATH_Injection)
- [OWASP WSTG: Testing for XPath Injection](https://owasp.org/www-project-web-security-testing-guide/latest/4-Web_Application_Security_Testing/07-Input_Validation_Testing/09-Testing_for_XPath_Injection)
- [PayloadsAllTheThings: XPATH Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/XPATH%20Injection)
