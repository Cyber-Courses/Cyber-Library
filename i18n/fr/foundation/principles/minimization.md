---
title: "Minimisation : garder moins, exposer moins, perdre moins"
description: "Le principe de sécurité qui consiste à réduire les données, les accès et la surface d'attaque au strict nécessaire pour un système."
keywords:
  - minimisation
  - minimisation des données
  - réduction de la surface d'attaque
  - moindre fonctionnalité
  - conservation des données
  - principe de minimisation
---

# Minimisation

**La minimisation est la discipline qui consiste à ne garder que ce dont on a besoin.** Chaque enregistrement qu'un système stocke, chaque porte qu'il laisse ouverte et chaque fonctionnalité qu'il livre est une chose qui peut être volée, détournée ou retournée contre vous. Le principe affirme que la version la plus sûre de chacun de ces éléments est celle qui n'existe pas : un système doit donc être réduit au minimum qui lui permet encore de faire son travail.

## Les trois choses qu'il vaut la peine de minimiser

La minimisation n'est pas une règle unique mais la même idée appliquée à trois types d'excès différents. Chacun a son propre responsable et son propre critère.

| Ce que vous minimisez | La question à poser | Ce qui le réduit |
|---|---|---|
| Données | Avons-nous vraiment besoin de collecter et de conserver cet enregistrement ? | Collecte ciblée, courtes durées de conservation, suppression par défaut |
| Accès | Qui et quoi peut atteindre ceci, et pour combien de temps ? | Moindre privilège, identifiants à portée limitée, autorisations limitées dans le temps |
| Surface d'attaque | Ce port, ce service ou cette fonctionnalité est-il réellement utilisé ? | Désactivation des fonctions inutilisées, suppression du code mort et des anciens systèmes |

La ligne relative aux accès fait l'objet de son propre principe. Cette page se concentre sur les deux éléments faciles à négliger : les données que vous détenez et la surface que vous exposez.

## "Tout collecter" contre "collecter ce dont on a besoin"

La minimisation des données a une filiation juridique autant que sécuritaire. Le droit de la vie privée la formule directement : le Règlement général sur la protection des données de l'Union européenne exige que les données personnelles soient "adéquates, pertinentes et limitées à ce qui est nécessaire" au regard de la finalité pour laquelle elles ont été collectées. Le cadre des États-Unis reprend cette idée ; les recommandations du NIST en matière de vie privée considèrent la minimisation de ce qui est collecté et conservé comme un moyen essentiel de réduire le risque.

La tension est réelle. Les équipes produit et analytique ont une tendance naturelle à "tout collecter", car des données qui existent aujourd'hui pourraient être utiles demain. La minimisation s'y oppose par un calcul simple : des données que vous n'avez jamais collectées ne peuvent pas fuiter, et des données que vous avez supprimées dans les temps ne sont pas dans la fuite. Un calendrier de conservation, où les enregistrements expirent automatiquement à moins qu'il y ait une raison de les garder, transforme ce principe en routine plutôt qu'en promesse.

## Réduire la surface d'attaque

La surface d'attaque est la somme de tous les points où un acteur extérieur pourrait tenter d'entrer : ports réseau ouverts, services en cours d'exécution, API exposées, champs de saisie, paquets installés et comptes utilisateurs. Sa minimisation est souvent appelée principe de moindre fonctionnalité, et le NIST SP 800-53 le nomme directement (contrôle CM-7) : configurer les systèmes pour qu'ils ne fournissent que les capacités essentielles et désactiver le reste.

La partie la plus dangereuse de la surface d'attaque est généralement celle dont personne ne se souvient. Un serveur de test, un ancien sous-domaine ou une page d'administration par défaut jamais désactivée reste sans surveillance et sans correctifs, offrant une voie d'entrée discrète.

## Cas pratique : le sous-domaine oublié

1. **Il est construit.** Une équipe met en place `beta.example.com` pour une démonstration, le fait pointer vers un hébergeur cloud et le charge d'une copie des données de production.
2. **Il est laissé à l'abandon.** La démonstration se termine. L'enregistrement DNS demeure, l'hôte continue d'exécuter une ancienne version logicielle, et personne ne l'ajoute à la liste des correctifs ou de la surveillance.
3. **Il est découvert.** Un attaquant qui analyse les domaines de l'entreprise trouve l'hôte oublié, exploite le service non corrigé et atteint les données d'exemple, qui se révèlent être de véritables enregistrements de clients.
4. **Ce que la minimisation aurait changé.** La copie des données n'aurait jamais dû exister (minimisation des données) ; l'hôte aurait dû être démantelé à la fin de la démonstration (surface d'attaque) ; et rien sur celui-ci n'aurait dû détenir d'identifiants de production (moindre privilège).

## Là où la minimisation renforce les autres

La minimisation est la partenaire discrète de plusieurs principes. Le [moindre privilège](least-privilege.md) applique la même logique aux permissions, en réduisant les accès plutôt que les données ou la surface. Une surface d'attaque plus réduite laisse à la [défense en profondeur](defense-in-depth.md) moins de brèches à couvrir, et elle soutient directement le [zero trust](zero-trust.md), qui suppose que tout service exposé peut être sondé. Décider de ce qu'il ne faut pas construire dès le départ est un geste central de la [sécurité dès la conception](security-by-design.md).

## Références

- [GDPR Article 5: Principles relating to processing of personal data](https://gdpr-info.eu/art-5-gdpr/)
- [NIST SP 800-53 Rev. 5 (CM-7 Least Functionality)](https://csrc.nist.gov/pubs/sp/800/53/r5/upd1/final)
- [NIST Privacy Framework](https://www.nist.gov/privacy-framework)
- [OWASP Cheat Sheet: Attack Surface Analysis](https://cheatsheetseries.owasp.org/cheatsheets/Attack_Surface_Analysis_Cheat_Sheet.html)
- [CISA: Secure by Design](https://www.cisa.gov/securebydesign)
