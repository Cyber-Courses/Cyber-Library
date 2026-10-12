---
title: "Applications : techniques offensives selon l'emplacement d'exécution du code"
description: "La catégorie applications se divise selon l'emplacement d'exécution du code, sur l'appareil de l'utilisateur, sur une infrastructure que l'opérateur contrôle, ou sur une infrastructure qu'un fournisseur exploite, car le modèle de confiance et la surface d'attaque atteignable diffèrent fondamentalement entre ces trois cas."
keywords:
  - application security
  - client-side attacks
  - server-side attacks
  - web application security
  - trust boundary
---

# Applications

La catégorie applications couvre le travail offensif contre les programmes qui mettent en œuvre le comportement d'une organisation, par opposition aux systèmes d'exploitation qui les hébergent (sa catégorie sœur sous [Software](../index.md)). C'est la partie la plus dense de la bibliothèque, car les applications exposent le plus de surface aux entrées non fiables et changent le plus souvent.

## Pourquoi cette division

Les trois sous-catégories se divisent selon l'emplacement d'exécution du code de l'application, ce qui détermine qui le contrôle et ce qu'un attaquant peut atteindre :

- **[Client](client/index.md)** : code qui s'exécute sur l'appareil de l'utilisateur (navigateurs, applications de bureau et mobiles). L'attaquant contrôle fréquemment l'environnement d'exécution, si bien que les questions portent sur ce à quoi le client fait confiance, les secrets qu'il détient, et la manière dont il peut être retourné contre son propre utilisateur ou utilisé pour atteindre le serveur.
- **[Serveur](server/index.md)** : code qui s'exécute sur une infrastructure que l'opérateur contrôle. L'attaquant ne l'atteint qu'à travers les interfaces qu'il expose, si bien que les questions portent sur la gestion des entrées, l'authentification et l'autorisation, et les services que le serveur assemble.
- **[Online](online/index.md)** : services qu'un fournisseur exploite (plateformes cloud et SaaS). Il n'y a pas de binaire atteignable à exploiter, si bien que l'attaquant travaille via le compte : identités, identifiants et jetons, OAuth et fédération, et la configuration du tenant atteinte via les propres API du fournisseur.

Cette division reflète une différence fondamentale de confiance. Côté client, l'attaquant possède souvent la machine, si bien que les protections ne sont qu'indicatives et que l'objectif est souvent d'influencer le serveur ou un autre utilisateur. Côté serveur, l'attaquant est distant et contraint à l'interface exposée, si bien que l'objectif est de faire faire à cette interface quelque chose qu'elle ne devrait pas faire. Une évaluation unique franchit généralement la frontière, puisque le comportement du client façonne ce qui atteint le serveur, mais les côtés client, auto-hébergé et hébergé par un fournisseur exigent chacun des hypothèses différentes, et c'est pourquoi la catégorie se ramifie ici avant d'aller plus en profondeur.

## Références

- [OWASP Top Ten](https://owasp.org/www-project-top-ten/)
- [OWASP Web Security Testing Guide](https://owasp.org/www-project-web-security-testing-guide/)
