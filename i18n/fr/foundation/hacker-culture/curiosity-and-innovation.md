---
title: "Curiosité et innovation : le moteur de la tradition hacker"
description: "Comment le besoin de comprendre et de détourner les systèmes a produit une grande partie des technologies qui définissent l'informatique moderne."
keywords:
  - curiosité des hackers
  - innovation en informatique
  - éthique hacker
  - bricolage et exploration
  - open source
  - histoire du hacking
---

# Curiosité et innovation

**La curiosité est la plus ancienne valeur de la culture hacker**, plus ancienne que tout manifeste, toute étiquette ou toute définition juridique. Bien avant que le mot "hacker" ne porte la moindre connotation criminelle, il désignait une personne qui voulait comprendre comment un système fonctionnait de l'intérieur, et qui considérait ce savoir comme quelque chose à étendre, partager et reconstruire. Le schéma se répète au fil des décennies : une personne rencontre une machine, refuse d'accepter les limites qu'on lui prête, et en explorant ces limites produit quelque chose que personne n'avait prévu.

## Des câblages à relais au PDP-1

Le mot remonte au Tech Model Railroad Club (TMRC) du MIT, à la fin des années 1950. Les membres qui entretenaient le réseau ferroviaire miniature du club, piloté par un système complexe de relais, employaient le mot "hack" pour désigner un travail astucieux et ludique, réalisé pour le plaisir et non pour une note ou un salaire. L'histoire complète des débuts de cette culture est racontée dans [Origines](origins.md).

Lorsque certains de ces étudiants ont eu accès aux premiers ordinateurs du MIT, d'abord le TX-0 puis un PDP-1 de DEC, cet état d'esprit est passé du câblage au code. Des enseignants comme Jack Dennis leur laissaient une grande liberté sur les machines, et c'est cette liberté d'expérimenter qui a transformé une habitude de club en tradition informatique.

## Trois projets que personne n'avait commandés

La preuve la plus claire que la curiosité alimente l'innovation se trouve dans ce qu'elle a produit alors que personne ne réclamait de produit. Chacun de ces projets a commencé par une question, et non par un cahier des charges.

| Projet | Lieu et époque | La question de départ | Ce qu'il est devenu |
| --- | --- | --- | --- |
| Spacewar! | MIT, PDP-1, 1961 à 1962 | Ce petit ordinateur pouvait-il faire tourner un jeu à deux joueurs, en temps réel, avec de la physique ? | L'un des premiers jeux vidéo influents, utilisé par DEC pour faire la démonstration de la machine |
| Unix et C | Bell Labs, 1969 à 1973 | À quoi ressemblerait un système d'exploitation plus simple et plus agréable ? | L'ancêtre de la plupart des systèmes d'exploitation de serveurs, d'ordinateurs de bureau et de mobiles utilisés aujourd'hui |
| Les RFC et les protocoles de l'ARPANET | UCLA et d'autres sites de l'ARPANET, à partir de 1969 | Comment les hôtes de ce nouveau réseau devaient-ils communiquer entre eux ? | Une méthode ouverte et publique d'élaboration des standards qui régit encore Internet |

Le cas de l'ARPANET illustre la dimension de partage de la curiosité. La RFC 1 de Steve Crocker (avril 1969) était explicitement une "request for comments", une demande de commentaires : un brouillon proposé ouvertement pour que d'autres puissent l'améliorer, plutôt qu'une spécification achevée imposée d'en haut.

## Les convictions de la génération du TMRC

Steven Levy, qui a écrit sur ces programmeurs du MIT dans *Hackers: Heroes of the Computer Revolution* (1984), a condensé leurs règles tacites sous le nom d'"éthique hacker". Plusieurs de ses principes sont de la curiosité érigée en règle :

- **L'impératif de la pratique.** On comprend un système en le démontant et en travaillant directement avec lui, pas seulement en lisant ce que ses concepteurs en disent.
- **Toute information doit être libre.** Le savoir se démultiplie lorsqu'il circule ; les techniques, le code source et les découvertes se transmettent donc plutôt que de s'accumuler jalousement.
- **Juger les hackers sur leur travail.** Une bonne solution tient par ses propres mérites, et non par les diplômes, l'âge ou le statut.
- **Les ordinateurs peuvent créer de l'art et de la beauté, et changer la vie en mieux.** Un code élégant mérite d'être recherché pour lui-même.

Sous tous ces principes repose une idée que les personnes décrites par Levy formulaient rarement de façon explicite : aucune conception n'est définitive, et chaque outil est une invitation à en construire une meilleure version. La manière dont ces valeurs se sont ensuite réparties entre des communautés distinctes fait l'objet de [Sous-cultures et valeurs](subcultures-and-values.md).

## Le réflexe du détournement en sécurité

Le détournement est le cœur pratique de la curiosité hacker : une technologie conçue dans un but est orientée vers un autre, et l'usage inattendu compte souvent plus que l'usage prévu. Un jeu devient une démonstration commerciale ; une expérience de laboratoire devient un système d'exploitation qui fait tourner une grande partie du monde.

Ce même réflexe anime aujourd'hui les logiciels open source et une grande partie du monde des start-up, et il est central dans la recherche en sécurité. Se demander "que peut faire d'autre cette entrée ?" ou "que se passe-t-il si j'envoie quelque chose que le concepteur n'avait jamais prévu ?" est précisément la façon dont les chercheurs trouvent les failles avant les attaquants. Un testeur qui ne vérifie que le comportement prévu d'un système passera à côté de l'essentiel.

## Quand la curiosité a besoin d'une autorisation

La curiosité, en elle-même, est neutre. La question qui conduit un chercheur à signaler une faille peut conduire quelqu'un d'autre à l'exploiter, et c'est pourquoi la voix de cette tradition s'est toujours accompagnée d'une réflexion éthique, comme dans le [Manifeste du hacker](hacker-manifesto.md). Ce qui sépare le travail constructif du préjudice n'est pas le degré de curiosité, mais le fait d'explorer avec une autorisation et avec considération pour les personnes qui dépendent du système. Cette distinction est à la base des catégories décrites dans [Chapeaux blancs, noirs et gris](white-black-and-grey-hats.md).

## Références

- [Steven Levy, Hackers: Heroes of the Computer Revolution, chapitres 1 et 2 (Project Gutenberg)](https://www.gutenberg.org/ebooks/729)
- [The Jargon File : entrée "hacker"](http://www.catb.org/jargon/html/H/hacker.html)
- [Computer History Museum : Spacewar! sur le PDP-1](https://www.computerhistory.org/pdp-1/spacewar/)
- [Dennis M. Ritchie : The Development of the C Language (Nokia Bell Labs)](https://www.nokia.com/bell-labs/about/dennis-m-ritchie/chist.html)
- [RFC 1 : Host Software, Steve Crocker (1969)](https://www.rfc-editor.org/rfc/rfc1)
- [Internet Society : Brief History of the Internet](https://www.internetsociety.org/internet/history-internet/brief-history-internet/)
