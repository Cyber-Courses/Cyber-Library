---
title: "Analyse de détection : corrélation et logique de détection"
order: 2
description: "Comment l'analyse transforme la télémétrie collectée en signaux pertinents grâce à la corrélation et à une logique de détection qui sépare l'activité réelle du bruit."
keywords:
  - analyse de détection
  - corrélation d'événements
  - logique de détection
  - signal et bruit
  - analytique de sécurité
  - détection d'anomalies
---

# Analyse

L'analyse est le travail qui consiste à transformer la télémétrie collectée en signaux pertinents. Elle applique de la corrélation et une logique de détection à de nombreux événements pour faire ressortir des motifs qui indiquent un comportement d'adversaire, en distinguant l'activité véritable de l'important volume de bruit bénin que génèrent les opérations normales.

Au sein de la phase de détection, l'analyse se situe entre la collecte brute et les alertes qui parviennent aux intervenants. Les lignes de journal prises isolément racontent rarement une histoire complète, c'est pourquoi l'analyse relie des événements à travers le temps, les hôtes et les identités pour révéler des séquences qui comptent : une connexion suspecte suivie d'une exécution de processus inhabituelle, ou un déplacement de données qui s'écarte d'une référence établie.

En pratique, l'analyse combine la correspondance par règles, l'établissement de références statistiques et l'analytique comportementale. Les défenseurs écrivent et affinent une logique de détection qui correspond à des techniques connues, règlent les seuils pour réduire les faux positifs et enrichissent les événements avec du contexte tel que la criticité des actifs et le renseignement sur les menaces. L'objectif est d'obtenir des signaux fiables et explicables sur lesquels un analyste peut agir en confiance, plutôt qu'un flot de correspondances de faible qualité. L'analyse s'améliore à mesure que les défenseurs apprennent quels motifs se révèlent pertinents et lesquels ne le sont pas.

## Références

- NIST SP 800-61, Computer Security Incident Handling Guide
- MITRE ATT&CK, Detections and Analytics
