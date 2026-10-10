---
title: "Données : manipulation offensive des encodages, chiffrements et transformations"
description: "La catégorie Données couvre les attaques sur la façon dont l'information est représentée et transformée, des encodages de caractères et formats binaires aux chiffrements et transformations réversibles, indépendamment de l'application qui la transporte."
keywords:
  - manipulation de données
  - attaques sur les encodages
  - attaques sur les chiffrements
  - données binaires
  - transformation de données
---

# Données

La catégorie Données couvre le travail offensif contre la façon dont l'information est représentée et transformée, en deçà du niveau d'une application quelconque. Les mêmes octets traversent encodeurs, chiffrements et sérialiseurs sur leur chemin entre systèmes, et chacune de ces étapes est une surface d'attaque à part entière : un décodeur qui fait confiance à son entrée, un chiffrement utilisé sans intégrité, ou une transformation réversible alors qu'on la supposait irréversible.

## Pourquoi ce découpage

Les sous-catégories regroupent par type de représentation ou de transformation attaquée, car les techniques et les outils diffèrent nettement de l'une à l'autre :

- **Encodage** : des schémas réversibles qui transportent les données sans secret (Base64, URL, pourcent, hexadécimal). L'angle offensif est le contournement de filtres, la contrebande et la confusion de parseurs.
- **Encodage de caractères** : la gestion des jeux de caractères et d'Unicode, où la normalisation, les formes surlongues et les homoglyphes mettent en échec comparaisons et validateurs.
- **Chiffrements** : des transformations cryptographiques, attaquées par leur mauvais usage plutôt que par les mathématiques, par exemple l'absence d'intégrité, les IV prévisibles et les comportements d'oracle.
- **Données binaires** : les formats binaires bruts et structurés, la manipulation au niveau hexadécimal et l'altération tenant compte du format.
- **Transformation de données** : la compression, la sérialisation et la conversion de format, où c'est la transformation elle-même (et non la charge utile) qui porte le défaut.

Découper selon la représentation garde chaque page centrée sur une classe de parseur ou d'algorithme, afin qu'une technique abusant de la normalisation Unicode ne se mêle pas à une autre abusant d'un mode de chiffrement par blocs. Ce sont des primitives fondamentales qui reviennent à travers les couches [logiciel](../software/index.md), [réseau](../network/index.md) et applicative, ce qui explique qu'elles vivent dans leur propre catégorie plutôt que sous une cible particulière.

## Références

- [OWASP: Testing for Weak Cryptography](https://owasp.org/www-project-web-security-testing-guide/latest/4-Web_Application_Security_Testing/09-Testing_for_Weak_Cryptography/)
- [OWASP: Injection and encoding](https://owasp.org/www-community/attacks/)
