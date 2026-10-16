---
title: "Mauvaise configuration : quand un logiciel sûr est déployé de façon non sécurisée"
description: "La mauvaise configuration est l'écart entre une conception sécurisée et un déploiement sécurisé, où les réglages par défaut et de petits oublis créent discrètement des ouvertures pour les attaquants."
keywords:
  - mauvaise configuration
  - configuration de sécurité défaillante
  - réglages par défaut
  - valeurs par défaut sécurisées
  - dérive de configuration
  - référentiel de durcissement
---

# Mauvaise configuration

La **mauvaise configuration** est la distance entre un système qui pourrait être sécurisé et celui qui l'est réellement, ouverte lorsqu'un logiciel est déployé avec des valeurs par défaut faibles, des fonctionnalités inutilisées laissées actives, ou des réglages qui n'ont jamais été resserrés. Le code peut être solide et la conception peut être robuste, et pourtant le système en fonctionnement laisse quand même fuir des données ou distribue des accès parce que quelqu'un a conservé une valeur d'usine, sauté une étape de durcissement, ou n'a pas compris ce qu'une option contrôlait. C'est l'une des causes profondes les plus fréquentes des véritables compromissions, et elle relève du facteur humain de la sécurité parce qu'elle naît de choix de déploiement plutôt que de défauts du logiciel lui-même.

## Du code correct, des réglages qui ne le sont pas

La façon la plus claire de comprendre la mauvaise configuration est de dire ce qu'elle n'est pas. Une [erreur de conception et de développement](design-and-development-errors.md) réside dans le logiciel, donc le correctif est une nouvelle version. Une mauvaise configuration réside dans la manière dont un produit existant et correct a été paramétré, donc le correctif est un réglage modifié, et non un correctif logiciel.

L'écart est large pour deux raisons. Les systèmes modernes exposent des milliers d'options ajustables à travers les systèmes d'exploitation, les bases de données, les services cloud et les applications, et une seule valeur erronée peut annuler tout le reste. Le problème se dissimule aussi très bien : un système mal configuré fonctionne généralement très bien pour les utilisateurs ordinaires, donc personne ne se plaint, et l'ouverture reste invisible jusqu'à ce que quelqu'un parte à sa recherche.

## Comment un réglage finit par être erroné

La plupart des mauvais réglages surviennent par l'une de quatre voies :

1. **Il n'a jamais été modifié.** Le produit a été livré ouvert, ou avec un mot de passe connu, et la personne qui l'a installé a accepté les valeurs par défaut.
2. **Il a été relâché pour faire fonctionner quelque chose.** Une permission a été élargie ou une vérification désactivée pendant un dépannage, puis jamais rétablie.
3. **Personne ne l'a compris.** Une option au nom peu clair ou à l'effet secondaire enfoui a été définie sans que personne ne connaisse son impact sur la sécurité.
4. **Il a dérivé.** Un système durci une fois s'est lentement éloigné de ce référentiel à mesure que les mises à niveau, les correctifs manuels et les nouveaux composants s'accumulaient.

## Ce qu'un réglage relâché livre

Le coût d'une valeur erronée est concret. Quelques cas courants :

| Réglage laissé erroné | Ce que gagne un attaquant |
|---|---|
| Identifiants par défaut ou fournis par l'éditeur | Une connexion directe à une console d'administration |
| Buckets de stockage ou bases de données publics | Téléchargement massif de données, souvent repéré par des scanners |
| Mode débogage, applications d'exemple, listage de répertoires | Chemins de code supplémentaires qui élargissent la [surface d'attaque](../concepts/assets-and-attack-surface.md) |
| Messages d'erreur verbeux | Chemins internes, versions et traces de pile qui facilitent l'étape suivante |

## Sécurisé par défaut et qui en porte le fardeau

La première voie, la valeur par défaut non modifiée, explique pourquoi l'initiative Secure by Design de la CISA demande aux éditeurs de livrer des produits verrouillés dès la sortie de la boîte, afin que l'état sûr soit celui que les clients obtiennent gratuitement. Cela recadre la responsabilité : lorsque la valeur par défaut n'est pas sûre, l'éditeur a pris une décision de conception qui pèse sur chaque installation. Lorsque la valeur par défaut est sûre, la mauvaise configuration devient une question plus étroite d'opérateurs qui s'en éloignent, ce qui s'accorde naturellement avec la [sécurité dès la conception](../principles/security-by-design.md).

## Là où elle rencontre les autres défaillances humaines

La mauvaise configuration est rarement isolée. Elle fait souvent suite au [comportement des utilisateurs et aux usages abusifs](user-behavior-and-misuse.md), comme lorsqu'un contournement précipité relâche discrètement un contrôle, et elle est atténuée par la [formation et la sensibilisation](training-and-awareness.md) qui apprennent aux opérateurs à remettre en question une valeur par défaut plutôt que de l'accepter.

## Références

- [OWASP Top 10: A05:2021 Security Misconfiguration](https://owasp.org/Top10/A05_2021-Security_Misconfiguration/)
- [NIST SP 800-53 Rev. 5 (Configuration Management family)](https://csrc.nist.gov/pubs/sp/800/53/r5/upd1/final)
- [CISA Secure by Design](https://www.cisa.gov/securebydesign)
- [CWE-1188: Initialization of a Resource with an Insecure Default](https://cwe.mitre.org/data/definitions/1188.html)
- [NIST SP 800-123: Guide to General Server Security](https://csrc.nist.gov/pubs/sp/800/123/final)
