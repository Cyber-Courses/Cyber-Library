---
title: "Erreurs de conception et de développement : des failles intégrées au système"
description: "Faiblesses de sécurité introduites lors de la conception et de la construction d'un système, de la logique défaillante aux valeurs par défaut fragiles en passant par les cas limites négligés."
keywords:
  - erreurs de conception et de développement
  - failles de sécurité logicielles
  - conception sécurisée
  - cas limites
  - validation des entrées
  - codage sécurisé
---

# Erreurs de conception et de développement

**Les erreurs de conception et de développement** sont les faiblesses de sécurité intégrées à un système pendant qu'il est planifié et construit, avant que quiconque n'en fasse un mauvais usage ou ne le configure mal en production. Ce ne sont pas des erreurs d'exploitation, mais des erreurs de construction. Une hypothèse erronée dans un schéma d'architecture, un morceau de logique qui gère le cas courant mais pas le cas inhabituel, ou une valeur par défaut pratique qui laisse une porte ouverte : tout cela devient partie intégrante du produit livré et le suit jusqu'à chaque installation.

## Failles du plan et bogues du code

Il est utile de distinguer deux niveaux où ceux qui construisent se trompent :

- **Les failles de conception** se trouvent dans le plan. Le système n'a jamais été prévu pour vérifier ce qu'il aurait dû vérifier, par exemple si la personne qui réinitialise un mot de passe est réellement titulaire du compte. L'OWASP le dit sans détour : une conception non sécurisée ne peut pas être corrigée par une implémentation parfaite, car le contrôle manquant n'a jamais été spécifié.
- **Les bogues d'implémentation** se trouvent dans le code. La conception était saine, mais un développeur a oublié une vérification de longueur, comparé des chaînes de la mauvaise manière ou inséré une saisie utilisateur dans une requête de base de données.

Cette distinction compte, car les remèdes diffèrent. Un bogue peut souvent être corrigé en quelques lignes. Une faille, comme faire confiance à des entrées auxquelles il ne faudrait jamais faire confiance, implique généralement de revoir la façon dont les composants communiquent entre eux, ce qui est lent et risqué une fois le produit entre les mains des clients.

Les deux partagent une propriété qui les distingue de la [mauvaise configuration](misconfiguration.md) ou du [mauvais usage par les utilisateurs](user-behavior-and-misuse.md) : ils sont livrés avec le produit. Une seule valeur par défaut fragile ou une seule erreur de logique atteint chaque client qui exécute le code, de sorte qu'une erreur unique est multipliée sur l'ensemble de la base d'utilisateurs.

## Six hypothèses qui deviennent des faiblesses

La plupart des erreurs de construction commencent par une hypothèse d'apparence raisonnable que personne n'a écrite ni remise en question.

| Erreur | Ce que le concepteur a supposé | Ce qui se passe réellement |
| --- | --- | --- |
| Validation des entrées absente ou faible | Les données arrivent dans la forme et la taille attendues | Des entrées malformées ou hostiles passent au travers et atteignent une logique incapable de les traiter |
| Cas limites négligés | Les valeurs seront ordinaires | Champs vides, nombres très grands, requêtes concurrentes ou encodages de caractères inhabituels se comportent d'une manière que personne n'a testée |
| Valeurs par défaut non sécurisées | Les utilisateurs durciront le produit après l'installation | Permissions ouvertes, mot de passe d'exemple ou chiffrement désactivé restent en l'état, car la plupart des utilisateurs ne les modifient jamais |
| Confiance erronée entre composants | Le service interne qui m'appelle est bienveillant | La compromission d'un composant se propage librement à tous les composants qui lui font confiance |
| Logique d'authentification maison | La connexion et les sessions sont assez simples pour les écrire nous-mêmes | Le code personnalisé gère mal l'expiration, la comparaison ou les états d'erreur tout en ayant l'air correct |
| Mélange des données et des instructions | Le texte de l'utilisateur n'est que de la donnée | Du texte collé directement dans une requête ou une commande modifie ce que fait le programme |

La Common Weakness Enumeration de MITRE répertorie en détail des centaines de ces schémas et constitue le vocabulaire habituel pour les nommer avec précision.

## Une fonction de réinitialisation de mot de passe, ratée de quatre façons

Prenons une seule fonctionnalité pour voir comment les deux niveaux d'erreur s'empilent. Une équipe doit permettre aux utilisateurs de réinitialiser un mot de passe oublié.

1. **La faille de conception.** La spécification indique : "l'utilisateur saisit son adresse e-mail, puis choisit un nouveau mot de passe sur l'écran suivant". Rien ne lie le second écran à une preuve de possession de l'adresse e-mail, de sorte que quiconque connaît une adresse peut prendre le contrôle du compte. Aucun codage soigneux ne peut corriger cela : l'étape manque dans le plan.
2. **Le cas limite.** Une fois la conception corrigée pour envoyer un lien de réinitialisation par e-mail, le jeton n'expire jamais et n'est pas invalidé après usage. Un vieux lien retrouvé dans une boîte de réception des mois plus tard fonctionne toujours.
3. **La valeur par défaut fragile.** Le jeton est généré avec une valeur prévisible, car la fonction aléatoire pratique du langage n'a pas été conçue pour la sécurité. Un attaquant peut deviner des jetons valides.
4. **Le bogue d'injection.** La recherche de l'adresse e-mail construit sa requête de base de données en concaténant l'adresse soumise, si bien qu'un "e-mail" forgé réécrit la requête.

Chaque étape correspond à un type d'erreur différent, introduit à un moment différent, et chacune aurait coûté bien moins cher à détecter sur le papier ou en revue qu'après la mise en production, où elle se transforme en cycle de correctifs, en divulgation coordonnée et souvent en incident.

## Détecter les erreurs au moment où elles sont commises

Les défenses les plus efficaces s'alignent sur les étapes où les erreurs apparaissent, une idée au cœur de la [sécurité dès la conception](../principles/security-by-design.md) et de l'initiative Secure by Design de la CISA, qui demande aux fabricants d'assumer les résultats en matière de sécurité de ce qu'ils livrent.

- **Pendant la conception : modéliser les menaces.** Se demander ce qui pourrait mal tourner tant que la conception n'est encore qu'un schéma et qu'un changement ne coûte qu'une discussion. C'est à ce stade qu'est détectée la vérification de titularité manquante dans l'exemple de réinitialisation.
- **Lors du choix des briques : réutiliser des composants éprouvés.** Privilégier des bibliothèques bien revues pour l'authentification, la cryptographie et l'analyse syntaxique plutôt que d'écrire les siennes.
- **Pendant l'écriture du code : valider à chaque frontière.** Traiter toutes les données externes comme non fiables, les vérifier par rapport à un format attendu strict et garder les données séparées des instructions (par exemple avec des requêtes paramétrées).
- **Lors du packaging : livrer des valeurs par défaut sécurisées.** Activer la configuration la plus sûre dès l'installation et obliger les utilisateurs à opter explicitement pour tout ce qui est plus risqué.
- **Avant la mise en production : tester l'abus, pas seulement l'usage.** Combiner revue de code, analyse statique et tests qui alimentent délibérément le système avec des entrées mauvaises, surdimensionnées et inattendues.

Ce sont des personnes qui commettent ces erreurs, donc les personnes font aussi partie de la solution : les développeurs qui connaissent les schémas de faiblesse courants sont bien moins susceptibles de les introduire (voir [formation et sensibilisation](training-and-awareness.md)). Et comme certaines erreurs passeront toujours au travers, le [moindre privilège](../principles/least-privilege.md) limite les dégâts que peut causer une seule faille.

## Références

- [OWASP Top Ten](https://owasp.org/www-project-top-ten/)
- [OWASP Top Ten 2021, A04: Insecure Design](https://owasp.org/Top10/A04_2021-Insecure_Design/)
- [MITRE CWE: Common Weakness Enumeration](https://cwe.mitre.org/)
- [NIST SP 800-160 Vol. 1 Rev. 1, Engineering Trustworthy Secure Systems](https://csrc.nist.gov/pubs/sp/800/160/v1/r1/final)
- [CISA: Secure by Design](https://www.cisa.gov/securebydesign)
- [OWASP Cheat Sheet Series: Input Validation](https://cheatsheetseries.owasp.org/cheatsheets/Input_Validation_Cheat_Sheet.html)
