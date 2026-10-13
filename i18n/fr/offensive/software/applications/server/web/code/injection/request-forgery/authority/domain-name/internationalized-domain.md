---
title: "Contournements par domaine internationalisé et normalisation"
description: "Les labels d'hôte Unicode et la normalisation se tiennent entre la chaîne qu'un filtre SSRF inspecte et le nom que le client résout, de sorte qu'un sosie qui se normalise vers un hôte bloqué, ou un validateur et un résolveur qui normalisent différemment, déjouent un contrôle basé sur les chaînes."
keywords:
  - domaine internationalisé
  - punycode
  - normalisation unicode
  - NFKC
  - contournement de blocklist SSRF
  - différentiel de validation de nom d'hôte
---

# Domaine internationalisé

Les noms de domaine internationalisés permettent aux noms d'hôtes de contenir de l'Unicode. Entre la chaîne qu'un validateur compare et le nom qu'un résolveur recherche se trouvent deux transformations : la normalisation Unicode (telle que NFKC) et la conversion IDNA `toASCII` vers le punycode. Les contournements pertinents pour le SSRF proviennent d'un décalage dans *quand* et *comment* ces transformations sont appliquées, et non d'un sosie à lui seul : un contrôle à correspondance exacte compare des chaînes, et un caractère confusable produit simplement une chaîne différente que la vérification rejette. Les cas utiles sont ceux où la normalisation fait correspondre une chaîne attaquante à un hôte qui intéresse la politique, ou où le validateur et le client normalisent différemment.

## Contournement de blocklist par normalisation

Le cas le plus fort est une **blocklist** basée sur un littéral ASCII tel que `localhost` ou un nom interne bloqué. La normalisation Unicode fait correspondre de nombreux caractères de compatibilité vers l'ASCII, donc un nom d'hôte qui n'est pas octet-pour-octet égal à la chaîne bloquée peut quand même se normaliser vers elle. Un validateur qui compare les octets bruts voit autre chose que `localhost` et l'autorise ; le client normalise ensuite et résout le nom bloqué, atteignant l'hôte interne :

```
ｌｏｃａｌｈｏｓｔ        # fullwidth Latin, NFKC-normalizes to localhost
ⓛⓞⓒⓐⓛⓗⓞⓢⓣ        # enclosed alphanumerics, NFKC-normalizes to localhost
```

La même chose s'applique à tout nom d'hôte interne qu'une blocklist tente d'exclure : fournissez une orthographe à caractères de compatibilité qui passe la vérification de chaîne et se normalise vers le nom interdit.

## Différentiel validateur / résolveur

Contre une **allowlist**, l'exigence est une différence dans la manière dont les deux étapes canonicalisent. Lorsque le validateur et le client HTTP appliquent une normalisation différente, un traitement IDNA différent, ou comparent sous des formes différentes (l'un Unicode, l'autre `xn--`), un seul nom d'hôte peut satisfaire la vérification tout en résolvant ailleurs. L'attaquant fournit l'hôte sous la forme que le validateur ne canonicalise pas de la même manière que le client :

```
аllowed.example      # a Cyrillic lookalike that a flawed validator folds toward the allowed label
xn--...              # or the A-label form, where the validator compares Unicode but the client resolves punycode
```

Cela ne fonctionne que là où une telle faille de canonicalisation existe ; sans elle, le sosie est un nom distinct que l'allowlist rejette. Le nom enregistré par l'attaquant doit aussi résoudre vers la cible interne, donc associez-le à une résolution que l'attaquant contrôle (ou au [DNS rebinding](dns-rebinding.md)).

## Les homographes sont visuels, pas un contournement à eux seuls

Un confusable tel qu'un `а` cyrillique (U+0430) pour le `a` latin fait qu'un nom d'hôte ressemble à un domaine autorisé, ce qui importe pour le phishing mais pas pour une comparaison de chaînes : l'homographe est un nom différent et une vérification exacte le rejette. Il n'aide au SSRF qu'en combinaison avec l'une des failles ci-dessus (une blocklist qui normalise, ou un différentiel validateur/résolveur). Traitez le sosie comme le vecteur de livraison et la faille de comparaison comme la vulnérabilité.

## Outils

- Confection manuelle de payloads de noms d'hôtes Unicode, punycode et à caractères de compatibilité.
- **curl** : soumettre chaque forme d'hôte confectionnée pour observer comment le client la normalise et la résout.

## Références

- [OWASP: Server Side Request Forgery](https://owasp.org/www-community/attacks/Server_Side_Request_Forgery)
- [Unicode Technical Standard 46: IDNA](https://www.unicode.org/reports/tr46/)
