---
title: "Curiosité et innovation : comment l'exploration a construit l'informatique moderne"
description: "Comment la volonté de comprendre et de détourner les systèmes a produit une grande partie de l'informatique moderne, et pourquoi ce même instinct est au coeur du travail de sécurité."
keywords:
  - curiosité du hacker
  - détournement de la technologie
  - histoire de l'innovation informatique
  - bricolage et exploration
  - découverte de vulnérabilités
  - éthique hacker
---

# Curiosité et innovation

**La curiosité est l'instinct fondateur de la culture hacker**, l'impulsion d'ouvrir un système, de comprendre comment il se comporte réellement, puis de le pousser au-delà de ce que ses concepteurs avaient prévu. Le schéma qui en résulte est remarquablement constant au fil des décennies : quelqu'un rencontre une machine, refuse d'accepter ses limites annoncées, et en testant ces limites produit quelque chose que personne n'avait conçu pour elle. Une grande partie de ce que nous appelons aujourd'hui l'informatique moderne a commencé exactement ainsi, comme un effet secondaire de personnes explorant pour le plaisir de comprendre.

## Ce que "hack" signifiait avant de signifier crime

Le mot précède sa réputation criminelle de plusieurs décennies. Au MIT, à la fin des années 1950, les membres du Tech Model Railroad Club utilisaient "hack" pour désigner un travail astucieux et ludique fait pour lui-même, pas pour une note ou un salaire. Le Jargon File, le dictionnaire de longue date de la communauté, définit encore un hacker d'abord comme "une personne qui aime explorer les détails des systèmes programmables et la manière d'étendre leurs capacités". Le crime apparaît très loin dans cette liste de sens, et le sens le plus ancien est de la pure curiosité. L'histoire complète de l'origine de cette attitude est racontée dans [Origines](origins.md).

## Le geste de détournement, en quatre inventions

La preuve la plus claire que la curiosité est le moteur de l'innovation est ce qu'elle produit lorsque personne n'a commandé de produit. Chacune des inventions suivantes est partie d'une question sur un système existant, pas d'un cahier des charges :

| Système | Conçu pour | Ce que la curiosité en a fait |
| --- | --- | --- |
| Le PDP-1 du MIT | Exécuter des programmes de laboratoire | Spacewar! (1962), l'un des premiers jeux vidéo influents |
| La recherche informatique des Bell Labs | Soutenir le travail interne | Unix et le langage C, ancêtres de Linux, macOS et Android |
| Le logiciel hôte d'ARPANET | Connecter des ordinateurs de recherche | Le processus ouvert des RFC qui régit encore les standards d'Internet |
| Le partage de documents du CERN | Organiser les articles de physique | Le World Wide Web |

Dans chaque ligne, le résultat a dépassé ce que quiconque avait d'abord imaginé, tantôt en étant orienté vers un objectif entièrement nouveau, comme le PDP-1 devenu un jeu, tantôt en atteignant un public bien plus vaste que celui pour lequel il avait été conçu, comme le Web. C'est cela, le geste d'exploration : une chose fabriquée ou sondée pour une raison devient la plus importante pour une autre, et la surprise devient la véritable contribution.

## Comment un "et si ?" devient un rapport de bug

Le même réflexe alimente la recherche en sécurité. Trouver une faille avant un attaquant, c'est de la curiosité appliquée, et cela suit généralement un chemin court :

1. **La question.** Un chercheur se demande ce qu'un système fait d'une entrée que son concepteur n'avait jamais anticipée.
2. **L'expérience.** Il envoie cette entrée inattendue et observe attentivement, au sein d'un système qu'il est autorisé à tester.
3. **La surprise.** Le système se comporte d'une manière que personne n'avait conçue, révélant une faiblesse.
4. **La divulgation.** La découverte est signalée afin qu'elle puisse être corrigée, plutôt qu'exploitée en silence.

Un testeur qui ne vérifie que le comportement promis par le manuel manquera presque tout ce qui compte. La curiosité est ce qui transforme une liste de contrôle en une véritable enquête.

## La même question, deux destinations

La curiosité elle-même est neutre. La question qui conduit une personne à rédiger un rapport de bug en conduit une autre à s'introduire par effraction, et c'est pourquoi la tradition hacker a toujours associé l'exploration à un débat sur l'éthique, exploré dans [savoir à double usage](../ethics-and-philosophy/dual-use-knowledge.md). Ce qui sépare la construction du préjudice n'est pas le degré de curiosité d'une personne, mais le fait qu'elle explore avec autorisation et avec un souci pour les gens qui dépendent du système. C'est exactement cette ligne que les catégories de [Chapeaux blancs, noirs et gris](white-black-and-grey-hats.md) tentent de tracer, et les valeurs qui se sont développées autour d'elle font l'objet de [Sous-cultures et valeurs](subcultures-and-values.md).

## Références

- [The Jargon File : entrée "hacker"](http://www.catb.org/jargon/html/H/hacker.html)
- [Steven Levy, Hackers: Heroes of the Computer Revolution (Project Gutenberg)](https://www.gutenberg.org/ebooks/729)
- [Computer History Museum : Spacewar! sur le PDP-1](https://www.computerhistory.org/pdp-1/spacewar/)
- [Dennis M. Ritchie : The Development of the C Language (Nokia Bell Labs)](https://www.nokia.com/bell-labs/about/dennis-m-ritchie/chist.html)
- [RFC 1: Host Software, Steve Crocker (1969)](https://www.rfc-editor.org/rfc/rfc1)
- [CERN : A short history of the Web](https://home.cern/science/computing/birth-web/short-history-web)
