---
title: "Règles de pare-feu : ouvrir le trafic entrant avec compute.firewalls"
order: 1
description: "Exposer ou atteindre des charges de travail en modifiant les règles de pare-feu VPC avec compute.firewalls.create ou update pour ouvrir le trafic entrant."
keywords:
  - règles de pare-feu
  - compute.firewalls
  - trafic entrant
  - VPC
  - exposition réseau
  - 0.0.0.0/0
---

# Règles de pare-feu

Les règles de pare-feu GCP sont des règles d'autorisation et de refus au niveau du VPC, mises en correspondance par direction, plage source, tags cibles ou comptes de service, et protocole/port. Avec `compute.firewalls.create` ou `compute.firewalls.update`, vous ouvrez un chemin vers une instance que vous souhaitez atteindre, et avec un accès en lecture, vous trouvez le trafic entrant que le projet laisse déjà ouvert vers internet.

## Trouver le trafic entrant grand ouvert

```bash
# every rule, with direction, source ranges, and allowed ports
gcloud compute firewall-rules list \
  --format="table(name,direction,sourceRanges.list(),allowed[].map().firewall_rule().list())"

# rules that allow the whole internet
gcloud compute firewall-rules list \
  --filter="sourceRanges:0.0.0.0/0 AND direction=INGRESS" \
  --format="table(name,allowed[].map().firewall_rule().list(),targetTags.list())"
```

Les règles par défaut `default-allow-ssh` et `default-allow-rdp` sur le réseau `default` en mode automatique exposent les ports `22` et `3389` sur internet pour chaque instance taguée.

## Ouvrir un chemin que vous contrôlez

```bash
# allow yourself to a target by tag or to the whole VPC
gcloud compute firewall-rules create allow-me \
  --network <vpc> --direction INGRESS --action ALLOW \
  --rules tcp:22,tcp:3389,tcp:3389 --source-ranges <your-ip>/32 \
  --target-tags <tag>
```

## Notes d'exploitation

- Les règles ciblent les instances par **tag réseau** ou **compte de service** ; une règle avec un tag cible étendu que de nombreuses instances portent les expose toutes à la fois.
- Un `deny` de haute priorité peut être masqué en créant un `allow` de numéro plus bas (priorité plus élevée), puisqu'une valeur de priorité plus basse l'emporte.
- Associez un trafic entrant exposé à [Compute Engine](../compute/compute-engine/index.md) pour confirmer quelles instances la règle expose et atteindre leurs métadonnées.

## Outils

- **gcloud** (`compute firewall-rules list`/`create`) : énumération et modifications de règles.
- **nmap** / **masscan** : confirmer l'accessibilité des ports exposés depuis l'extérieur.

## Références

- [HackTricks Cloud : pare-feu et réseau GCP](https://cloud.hacktricks.wiki/en/pentesting-cloud/gcp-security/index.html)
- [Google Cloud : règles de pare-feu VPC](https://cloud.google.com/firewall/docs/firewalls)
- [Hacking the Cloud : GCP](https://hackingthe.cloud/)
