---
title: "Chasseur de menaces : spécialiste de la détection proactive des adversaires"
description: "Un chasseur de menaces recherche de manière proactive, dans les réseaux et les systèmes, les menaces qui échappent aux défenses automatisées, en formulant des hypothèses sur le comportement des adversaires et en les confrontant à la télémétrie."
keywords:
  - chasse aux menaces
  - sécurité défensive
  - ingénierie de détection
  - comportement des adversaires
  - télémétrie des terminaux
  - mitre att&ck
---

# Chasseur de menaces

Plutôt que d'attendre qu'une alerte se déclenche, un **chasseur de menaces** part du principe qu'un adversaire compétent est peut-être déjà présent et s'attache à prouver ou à réfuter cette hypothèse. Le rôle repose sur un postulat simple : les outils basés sur les signatures et l'alerte pilotée par des règles ne peuvent pas tout attraper, et la curiosité humaine éclairée par la connaissance du savoir-faire des attaquants peut faire émerger des intrusions que les défenses manquent. Chaque chasse développe aussi une compréhension analytique approfondie de la manière dont les adversaires opèrent et la transforme en améliorations durables de la couverture de détection d'une organisation.

## Responsabilités

- Formuler des **hypothèses**, articulées autour d'une technique d'adversaire précise, d'un schéma d'authentification suspect ou d'un usage anormal d'un outil d'administration légitime.
- Tester chaque hypothèse en interrogeant la télémétrie de détection et de réponse sur les terminaux, les enregistrements de flux réseau, les journaux d'authentification et les pistes d'audit du cloud.
- Mener des chasses structurées guidées par des cadres tels que **MITRE ATT&CK**, aux côtés d'une analyse exploratoire suscitée par du renseignement sur les menaces récent.
- Transmettre l'activité malveillante confirmée à la réponse à incident pour confinement et correction.
- Convertir ce qu'une chasse révèle en une nouvelle règle de détection, comblant la faille qui a laissé l'activité passer inaperçue.

## Compétences et profil

- **Connaissance de l'environnement :** les rouages internes des systèmes d'exploitation, les sources de journalisation courantes, et la manière dont le comportement administratif normal diffère de l'abus des mêmes outils.
- **Aisance avec les données :** écrire des requêtes dans des langages propres à chaque plateforme, travailler avec de grands volumes de données et raisonner sur ce qui est rare, attendu ou hors de propos.
- **Savoir-faire des adversaires :** une familiarité avec les tactiques, techniques et procédures, généralement organisées via le cadre ATT&CK, pour donner une structure à la recherche.
- **Honnêteté intellectuelle :** la discipline d'abandonner une hypothèse lorsque les données ne la soutiennent pas.
- **Communication :** des constats écrits clairs et des notes méthodiques, afin que les intervenants et la direction puissent agir sur ce qu'une chasse révèle.

## Accéder au poste

Peu de personnes débutent directement ici ; la plupart arrivent après avoir bâti des fondations dans les opérations défensives. Un parcours courant va de la surveillance et du triage des alertes vers le travail d'investigation, où un analyste développe la connaissance de l'environnement et la familiarité avec les outils dont dépend la chasse. L'expérience en ingénierie de détection, en investigation numérique ou en renseignement sur les menaces est aussi une rampe d'accès naturelle, puisque chacune alimente les hypothèses qu'un chasseur poursuit.

## Évolution de carrière

La progression va vers davantage d'autonomie et d'influence : définir la stratégie d'un programme de chasse, encadrer les analystes plus récents, construire de l'outillage et du contenu de détection réutilisables, et traduire les résultats des chasses en améliorations mesurables de la couverture. Les chasseurs seniors façonnent souvent la manière dont toute une organisation de sécurité priorise ses efforts de détection.

## Rôles associés

- [Analyste SOC](soc-analyst.md) : répond aux alertes à mesure qu'elles arrivent, là où le chasseur travaille en amont de l'alerte, cherchant ce qu'aucune règle n'a encore signalé.
- [Analyste de malwares](malware-analyst.md) : décortique un artefact précis, tandis que le chasseur se sert de cette compréhension pour rechercher le comportement à travers l'environnement.
- [Intervenant en réponse à incident](../forensics-and-ir-roles/incident-responder.md) : dirige le confinement et la reprise une fois une intrusion confirmée, souvent une intrusion que le chasseur a fait émerger en premier lieu.

## Références

- [NICE Workforce Framework for Cybersecurity (NIST SP 800-181)](https://csrc.nist.gov/pubs/sp/800/181/r1/final)
- [MITRE ATT&CK (attack.mitre.org)](https://attack.mitre.org/)
- [SANS Institute, ressources et cursus sur la chasse aux menaces](https://www.sans.org/)
