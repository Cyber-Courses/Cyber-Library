---
title: "Renseignement tactique sur les menaces : TTP et détection défensive"
description: "Comment le renseignement tactique décrit les tactiques, techniques et procédures des adversaires pour que les défenseurs de première ligne puissent détecter et réagir. Une vue d'ensemble de référence au sein des quatre niveaux du renseignement sur les cybermenaces."
keywords:
  - renseignement tactique sur les menaces
  - tactiques techniques procédures
  - comportement des adversaires
  - renseignement sur les menaces pour le soc
  - ingénierie de détection
  - renseignement de défense de première ligne
---

# Renseignement tactique

Le renseignement tactique décrit la manière dont les adversaires opèrent : les tactiques, techniques et procédures qu'ils utilisent pour obtenir un accès, se déplacer dans un environnement et atteindre leurs objectifs. Il fournit aux défenseurs de première ligne le détail comportemental nécessaire pour construire des détections et façonner la réponse. Il se situe juste au-dessus du renseignement technique, qui fournit les indicateurs les plus atomiques et les plus éphémères, comme les empreintes (hashes), les adresses et les domaines. Son public est le centre des opérations de sécurité, les ingénieurs de détection et les analystes qui agissent en minutes et en heures plutôt qu'en semaines.

Parmi les quatre niveaux de renseignement, le travail tactique figure parmi les couches les plus proches du clavier. Il alimente les règles de détection, les hypothèses de chasse et les décisions de triage, et il est le plus utile lorsqu'il s'exprime sous forme de comportement adverse reproductible plutôt que d'artefacts ponctuels. Des cadres comme MITRE ATT&CK aident les analystes à organiser les techniques de manière cohérente, ce qui facilite la mise en correspondance du renseignement tactique avec la couverture défensive et son partage entre les équipes.

Le renseignement tactique est important parce qu'il raccourcit le délai entre l'action d'un adversaire et la réponse d'un défenseur, et parce qu'un comportement est plus difficile à changer pour un adversaire qu'un indicateur isolé. Il soutient une détection plus durable, un triage plus précis et une meilleure priorisation des alertes. Ses sources comprennent la télémétrie, l'analyse des logiciels malveillants et de l'outillage, les rapports d'incident et les bibliothèques structurées de techniques. Sa principale limite est que les techniques évoluent encore et peuvent être dissimulées, de sorte que le renseignement tactique fonctionne mieux lorsqu'il est rattaché au contexte plus large de la campagne décrit au niveau opérationnel et enrichi des indicateurs concrets issus du niveau technique.

## Références

- Documentation du cadre MITRE ATT&CK
- FIRST Traffic Light Protocol et recommandations de partage d'information
