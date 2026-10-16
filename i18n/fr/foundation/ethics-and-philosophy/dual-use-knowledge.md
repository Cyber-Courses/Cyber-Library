---
title: "Connaissance à double usage : la compétence qui défend est celle qui nuit"
description: "La connaissance à double usage est une information ou une technique qui peut défendre ou nuire selon qui la détient et pourquoi il l'utilise."
keywords:
  - connaissance à double usage
  - technologie à double usage
  - divulgation coordonnée des vulnérabilités
  - sécurité par l'obscurité
  - sécurité offensive
  - contrôles à l'exportation
---

# Connaissance à double usage

**La connaissance à double usage est une information ou une compétence qui protège et qui met en danger avec la même facilité, de sorte que son poids moral ne réside pas dans la connaissance mais dans la main qui la détient.** Apprendre comment crocheter une serrure vous enseigne à la fois à la déjouer et à en choisir une meilleure. En sécurité, presque chaque capacité est construite ainsi : la compréhension qui permet à un défenseur de trouver une faille est la même que celle qui permet à un attaquant de l'atteindre.

## L'origine d'un terme chargé dans le contrôle à l'exportation

L'expression "double usage" n'est pas née avec les ordinateurs. Elle vient du contrôle des armements, où un bien à double usage est une technologie, comme une centrifugeuse, un précurseur chimique ou une souche de pathogène, ayant à la fois des usages civils et militaires. L'Arrangement de Wassenaar, un régime multilatéral de contrôle à l'exportation fondé en 1996, existe pour suivre ces biens lorsqu'ils franchissent les frontières. En 2013, sa liste de contrôle a été élargie pour couvrir les systèmes et outils servant à produire, commander et livrer des "logiciels d'intrusion", une tentative de traiter ces boîtes à outils de surveillance et d'intrusion comme des armes, même si les charges d'exploit elles-mêmes restaient hors du champ du contrôle. La réaction des chercheurs fut immédiate : le même code qu'un gouvernement pourrait acheter à des fins de surveillance est celui qu'un défenseur utilise pour tester et corriger, de sorte qu'une règle écrite pour arrêter l'un bloque l'autre. L'épisode a montré à quel point la catégorie est difficile à réguler quand l'objet est de la connaissance plutôt que du matériel.

## La même technique dans deux mains

Aucune colonne ci-dessous ne contient une compétence différente. Le balayage de ports reste du balayage de ports. Ce qui change, c'est qui l'exécute, contre quoi, et avec la permission de qui.

| Technique | Dans les mains d'un défenseur | Dans les mains d'un attaquant |
| --- | --- | --- |
| Recherche de vulnérabilités | Trouve les failles pour qu'elles soient corrigées | Trouve les failles pour qu'elles soient exploitées |
| Code d'exploit | Prouve qu'un risque est réel et amorce des détections | Transforme une faille en intrusion opérationnelle |
| Rétro-ingénierie | Extrait des indicateurs de maliciels | Copie les méthodes d'un rival pour les réutiliser |
| Test d'identifiants | Mesure la faiblesse des mots de passe | Vole l'accès à un compte |

Parce que la compétence ne change jamais, toute la question se réduit à la permission et à l'intention, raison pour laquelle [l'autorisation et le consentement](authorization-and-consent.md) pèsent tant dans ce domaine.

## Mener une vulnérabilité jusqu'au public

Prenez un chercheur qui découvre une faille grave dans un logiciel largement utilisé. La découverte est désormais à double usage au sens le plus aigu, et ce qui se passe ensuite décide qui elle aide.

1. **La thésauriser.** Gardée secrète ou vendue, la faille sert uniquement celui qui la détient, et le public reste exposé sans le savoir.
2. **Coordonner la divulgation.** Le chercheur prévient d'abord l'éditeur, convient d'un calendrier et attend un correctif avant que les détails ne deviennent publics. Les défenseurs y gagnent ; les attaquants sont retardés.
3. **Publier intégralement.** Tout publier d'un coup arme défenseurs et attaquants au même instant, selon la théorie que la pression publique force une réparation plus rapide.

Il n'existe aucune voie qui n'aide que les acteurs bienveillants. Cette absence est le problème qui définit le domaine.

## Pourquoi le secret n'est pas une valeur par défaut sûre

La réponse tentante est simplement de verrouiller la connaissance. L'histoire va à l'encontre de ce choix. Garder les méthodes secrètes, la posture connue sous le nom de sécurité par l'obscurité, tend à abriter les attaquants qui redécouvriront la faille de toute façon tout en laissant aveugles les défenseurs honnêtes. La connaissance ouverte est inconfortable mais généralement plus sûre, un jugement qui se heurte directement à [la liberté face au contrôle](freedom-versus-control.md) et à la pesée plus large dans [la sécurité face à la vie privée](security-versus-privacy.md). Décider où se situe cette ligne, et répondre du résultat, relève de la [responsabilité morale](moral-responsibility.md), et non d'une règle que quiconque peut vous remettre.

## Références

- [The Wassenaar Arrangement](https://www.wassenaar.org/)
- [ISO/IEC 29147:2018, Vulnerability disclosure](https://www.iso.org/standard/72311.html)
- [CISA Coordinated Vulnerability Disclosure Process](https://www.cisa.gov/coordinated-vulnerability-disclosure-process)
- [EFF Coders' Rights Project](https://www.eff.org/issues/coders-rights)
