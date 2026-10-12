---
title: "Journalisation et détection GCP"
order: 8
description: "Aveugler la détection GCP : falsifier les sinks Cloud Logging, désactiver les journaux d'audit d'accès aux données et affaiblir Security Command Center."
keywords:
  - Cloud Logging
  - journaux d'audit
  - sink de journaux
  - Security Command Center
  - évasion de détection
  - anti-forensique
---

# Journalisation et détection

GCP enregistre l'activité sur deux couches : les **Cloud Audit Logs** capturent les appels d'API, et **Cloud Logging** achemine tout à travers des sinks de journaux vers des buckets et des destinations externes, avec **Security Command Center** qui remonte des findings par-dessus. Un opérateur dégrade ces éléments pour agir sans être vu, choisissant entre le brutal (supprimer un sink, désactiver un type de journal) et le discret (un filtre d'exclusion qui ne retire que vos événements), en pesant ce qui s'enregistre encore au niveau de l'organisation au-dessus du projet.

## Ce qui est regroupé ici

- **[Cloud Logging](cloud-logging.md)** : supprimer ou détourner les sinks de journaux et ajouter des filtres d'exclusion pour retirer l'activité des journaux.
- **[Journaux d'audit](audit-logs.md)** : désactiver ou restreindre les Cloud Audit Logs, en particulier les journaux d'accès aux données, pour masquer les appels d'API.
- **[Security Command Center](security-command-center.md)** : mettre en sourdine ou désactiver les findings et les sources pour supprimer les alertes.

## Références

- [HackTricks Cloud : GCP](https://cloud.hacktricks.wiki/en/pentesting-cloud/gcp-security/index.html)
- [Stratus Red Team : techniques GCP](https://stratus-red-team.cloud/attack-techniques/GCP/)
- [Hacking the Cloud : GCP](https://hackingthe.cloud/)
