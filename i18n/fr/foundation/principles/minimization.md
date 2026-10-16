---
title: "Minimisation : ne garder que le strict nécessaire"
description: "Le principe de sécurité consistant à réduire les données, les accès et la surface d'attaque pour qu'un système ne détienne et n'expose que ce dont il a réellement besoin."
keywords:
  - minimisation
  - minimisation des données
  - réduction de la surface d'attaque
  - fonctionnalité minimale
  - protection de la vie privée dès la conception
  - principes de sécurité
---

# Minimisation

**La minimisation** est la pratique consistant à garder un système aussi réduit que possible tout en lui permettant de remplir sa fonction : moins de données stockées, moins d'accès accordés, et moins de fonctionnalités, d'interfaces et de composants exposés. Le raisonnement est direct. Ce qu'un système ne détient pas ne peut pas être volé, ce qu'il n'expose pas ne peut pas être attaqué, et ce qu'il ne permet pas ne peut pas être détourné. La minimisation se retrouve dans tout le travail de sécurité et de protection de la vie privée, depuis la réglementation sur la protection des données qui impose de ne collecter que ce qui est nécessaire jusqu'aux guides de durcissement qui suppriment les services inutilisés d'un serveur.

## Trois choses à réduire

La minimisation est une seule idée appliquée à trois types d'"éléments" différents. Chacun a sa propre question directrice et ses propres mesures habituelles.

| Ce que l'on réduit | La question à se poser | Mesures habituelles |
| --- | --- | --- |
| **Données** | Cette finalité a-t-elle réellement besoin de ce champ, et pour combien de temps ? | Ne collecter que les champs requis, fixer des durées de conservation, supprimer ou archiver une fois la finalité atteinte |
| **Fonctionnalités** | Ce service, ce protocole, ce compte ou ce paquet doit-il vraiment être actif ? | Désinstaller les logiciels inutilisés, désactiver le code d'exemple et les fonctions de débogage, supprimer les comptes par défaut |
| **Exposition** | Qui a besoin d'atteindre cette interface, et depuis où ? | Fermer les ports inutilisés, publier l'ensemble le plus restreint d'API et de points de terminaison, garder les systèmes internes hors du réseau public |

Le bénéfice diffère légèrement pour chacun. Moins de données signifie qu'une compromission expose moins : une base de données qui ne conserve que les enregistrements actuels et nécessaires divulgue bien moins qu'une base qui détient des années de données personnelles inutilisées. Moins de fonctionnalités et moins d'exposition signifient une surface d'attaque plus petite, car chaque port ouvert, paquet installé, point de terminaison d'API et fonctionnalité activée est une voie d'entrée potentielle. Les trois réduisent le coût et la complexité, puisqu'il y a moins de choses à corriger, surveiller, sauvegarder et comprendre.

## Se demander "en avons-nous besoin ?" tout au long du cycle de vie des données

La minimisation des données n'est pas une décision unique prise au moment de la collecte. La même question revient à chaque étape que traverse une donnée :

1. **Collecte.** Ne recueillir que les champs dont le traitement a véritablement besoin. Résister au champ "au cas où il servirait plus tard" ; il devient un passif bien avant de devenir utile.
2. **Utilisation et accès.** Donner à chaque personne et à chaque service l'accès à la portion de données la plus étroite qu'exige sa tâche, afin que même un accès accordé atteigne le moins de choses possible.
3. **Stockage.** Privilégier les formes qui contiennent moins : stocker une valeur dérivée ou un identifiant tronqué plutôt que l'original complet lorsque cela suffit.
4. **Partage et copie.** Chaque export, flux d'analyse ou intégration tierce est une nouvelle copie qui doit être minimisée à part entière.
5. **Suppression.** Décider dès la conception quand les données seront supprimées et construire le mécanisme qui s'en charge. Des données collectées sans plan de suppression ne font que s'accumuler.

## Où se cachent les copies et fonctionnalités superflues

Les systèmes deviennent rarement surchargés à cause d'une seule grande décision. Ils grossissent par des habitudes et des canaux secondaires qui échappent à la revue appliquée à la conception principale :

- **Les réglages par défaut qui collectent tout :** les équipes recueillent des données par défaut, en supposant que plus est toujours mieux, et constituent un stock qu'elles n'utilisent jamais mais qu'elles doivent tout de même protéger.
- **Les copies fantômes :** les sauvegardes, journaux, exports et pipelines d'analyse dupliquent discrètement des données sensibles en dehors des systèmes qui avaient été soigneusement minimisés.
- **La conservation oubliée :** les stocks continuent de grossir parce qu'aucune tâche ne supprime jamais rien.
- **La prolifération des fonctionnalités en production :** des modules optionnels, des points de terminaison de débogage et du code d'exemple sont livrés activés parce que personne ne les a désactivés.

Une base de données principale minimisée protège peu si un fichier journal nocturne contient les mêmes enregistrements en intégralité. C'est souvent en examinant ces canaux secondaires que la véritable réduction se produit.

## De la réglementation aux référentiels de durcissement

La minimisation est inscrite à la fois dans le droit de la protection de la vie privée et dans les catalogues de contrôles de sécurité, ce qui explique en partie qu'elle apparaisse dans des contextes si différents :

- **Droit de la protection de la vie privée.** Le Règlement général sur la protection des données de l'UE (RGPD) exige, à l'article 5, que les données personnelles soient "adéquates, pertinentes et limitées à ce qui est nécessaire" au regard des finalités (minimisation des données) et conservées pas plus longtemps que nécessaire (limitation de la conservation). Des exigences similaires existent dans de nombreuses autres juridictions et constituent un élément central de la protection de la vie privée dès la conception.
- **Contrôles de sécurité.** NIST SP 800-53 comprend le contrôle CM-7, Least Functionality, qui demande aux organisations de configurer les systèmes pour qu'ils ne fournissent que les capacités essentielles et d'interdire ou de restreindre les fonctions, ports, protocoles et services inutiles.
- **Analyse de la surface d'attaque.** Des guides tels que l'OWASP Attack Surface Analysis Cheat Sheet considèrent la cartographie puis la réduction des points d'entrée comme une partie courante de la sécurité applicative.

## La minimisation aux côtés des autres principes

La minimisation fonctionne rarement seule. Le [moindre privilège](least-privilege.md) applique aux permissions le même raisonnement que la minimisation applique aux données et à la surface d'attaque, et les deux sont généralement conçus ensemble. Dans une stratégie de [défense en profondeur](defense-in-depth.md), un système plus réduit donne à chaque couche moins de choses à protéger et moins de brèches à couvrir. La minimisation est aussi l'un des tout premiers choix faits dans le cadre de la [sécurité dès la conception](security-by-design.md), car retirer un composant après coup est bien plus difficile que de ne jamais l'ajouter. Enfin, le [Zero Trust](zero-trust.md) la complète : limiter ce qui est exposé va naturellement de pair avec la vérification de chaque requête au regard du plus petit accès nécessaire. Pour la notion plus large de ce qu'un attaquant peut atteindre, voir [actifs et surface d'attaque](../concepts/assets-and-attack-surface.md).

## Références

- [NIST SP 800-53 Rev. 5, Security and Privacy Controls (CM-7 Least Functionality)](https://csrc.nist.gov/pubs/sp/800/53/r5/upd1/final)
- [NIST Glossary: Attack Surface](https://csrc.nist.gov/glossary/term/attack_surface)
- [OWASP: Attack Surface Analysis Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Attack_Surface_Analysis_Cheat_Sheet.html)
- [Règlement (UE) 2016/679, Règlement général sur la protection des données (EUR-Lex)](https://eur-lex.europa.eu/eli/reg/2016/679/oj)
- [UK Information Commissioner's Office : principe de minimisation des données](https://ico.org.uk/for-organisations/uk-gdpr-guidance-and-resources/data-protection-principles/a-guide-to-the-data-protection-principles/data-minimisation/)
