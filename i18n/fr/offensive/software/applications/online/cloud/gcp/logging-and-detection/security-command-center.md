---
title: "Security Command Center : mettre en sourdine et désactiver les findings"
order: 3
description: "Mettre en sourdine ou désactiver les findings et les sources de Security Command Center pour supprimer les alertes."
keywords:
  - Security Command Center
  - SCC
  - findings
  - mise en sourdine
  - évasion de détection
  - alertes
---

# Security Command Center

Security Command Center (SCC) est l'agrégateur de findings de GCP : il remonte des alertes depuis Event Threat Detection, Security Health Analytics et les sources connectées, et les achemine ensuite via des configurations de notification. Mettre des findings en sourdine, écrire une règle de mise en sourdine qui pré-silence votre activité, ou démanteler la configuration de notification empêche l'alerte d'atteindre les défenseurs. Les permissions SCC se trouvent sous `securitycenter.*` au scope de l'organisation ou du projet.

## Mettre en sourdine ce qui est déjà remonté

```bash
gcloud scc findings list <org-or-project> --filter="category=\"...\""
gcloud scc findings set-mute <finding> --organization=<org> --mute=MUTED
```

## Pré-silencer avec une règle de mise en sourdine

```bash
# a mute config auto-mutes matching future findings (e.g. your IPs or SA)
gcloud scc muteconfigs create attacker-mute \
  --organization=<org> \
  --filter='resource.project_display_name="prod" AND finding_class="THREAT"'
```

## Couper le chemin de notification

```bash
gcloud scc notifications list --organization=<org>
gcloud scc notifications delete <config> --organization=<org>   # stop Pub/Sub alerting
```

## Notes d'exploitation

- SCC est principalement un service au **scope de l'organisation** ; un principal limité au projet ne peut souvent pas mettre en sourdine les findings au niveau de l'organisation, confirmez donc le scope qu'atteint votre jeton.
- La mise en sourdine ne supprime pas un finding, elle le masque de la vue par défaut ; un défenseur filtrant sur les findings en sourdine le voit toujours, une règle de mise en sourdine est donc une suppression visuelle, pas un effacement.
- Supprimer la configuration de notification arrête la poussée vers Pub/Sub et le SIEM en aval tout en laissant les findings dans la console, ce qui est plus discret que de désactiver entièrement les détecteurs.

## Outils

- **gcloud** (`scc findings`, `scc muteconfigs`, `scc notifications`) : tout ce qui précède.
- **ScoutSuite** / **Prowler** : énumérer les sources SCC et les configurations de notification.

## Références

- [HackTricks Cloud : GCP](https://cloud.hacktricks.wiki/en/pentesting-cloud/gcp-security/index.html)
- [Stratus Red Team : techniques GCP](https://stratus-red-team.cloud/attack-techniques/GCP/)
- [Google : mettre en sourdine les findings de Security Command Center](https://cloud.google.com/security-command-center/docs/how-to-mute-findings)
