---
title: "Matériel : techniques offensives sur toute la pile matérielle"
description: "La catégorie Matériel organise les attaques selon la couche de la pile matérielle qu'elles visent, du silicium et de la carte jusqu'au micrologiciel, à la plateforme et à l'architecture."
keywords:
  - hacking matériel
  - attaques sur micrologiciel
  - sécurité du silicium
  - attaques au niveau carte
  - architecture matérielle
---

# Matériel

La catégorie Matériel couvre le travail offensif contre les dispositifs informatiques physiques et le code de bas niveau qui s'exécute au plus près d'eux. Ces attaques supposent un accès physique ou quasi physique à un appareil et exploitent le fait que les frontières de confiance s'affaiblissent à mesure qu'on descend : un secret protégé par logiciel peut souvent être lu directement sur un bus, une puce flash ou un port de débogage.

## Pourquoi ce découpage

Les sous-catégories suivent les couches de la pile matérielle, car l'accès, l'outillage et les primitives changent à chaque couche :

- **Silicium** : la puce elle-même, y compris les canaux auxiliaires, l'injection de fautes et les attaques sur les secrets intégrés et les éléments sécurisés.
- **Carte** : le circuit imprimé et ses interconnexions, où vivent l'écoute de bus, les interfaces de débogage (JTAG, SWD) et la récupération par retrait de puce.
- **Micrologiciel** : le code stocké sur l'appareil (BIOS/UEFI, micrologiciels embarqués, images de microcontrôleurs), extrait, analysé et modifié.
- **Plateformes** : des classes d'appareils intégrées et leurs écosystèmes propres, où l'attaque combine plusieurs couches contre un produit réel.
- **Architecture** : les propriétés au niveau de la conception (jeux d'instructions, modèles mémoire, exécution de confiance) qui façonnent ce que les couches inférieures exposent.

Organiser par couche correspond au déroulement réel d'une évaluation, de l'extérieur vers l'intérieur : identifier la plateforme, extraire le micrologiciel, sonder la carte et n'atteindre le silicium que lorsque les couches supérieures ne cèdent pas. Chaque couche demande un équipement différent et un modèle mental différent, si bien que les garder séparées empêche qu'une technique d'analyseur logique soit classée à côté d'une technique d'analyse de puissance. Cette catégorie se distingue de la sécurité [physique](../physical/index.md), qui consiste à pénétrer des espaces et à déjouer des barrières plutôt qu'à attaquer des appareils.

## Références

- [OWASP Firmware Security Testing Methodology](https://github.com/scriptingxss/owasp-fstm)
- [NIST: Hardware Security](https://csrc.nist.gov/)
