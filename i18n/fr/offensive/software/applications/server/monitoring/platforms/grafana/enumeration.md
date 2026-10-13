---
title: "Énumération : identifier Grafana"
order: 4
description: "Grafana expose sa version sur la page de connexion, l'endpoint /api/health et les ressources statiques, généralement sur le port 3000. La version est décisive car la traversée de chemin non authentifiée et d'autres vulnérabilités sont spécifiques à la version, de sorte que lire le build exact oriente le choix de l'exploit avant toute authentification."
keywords:
  - grafana enumeration
  - api/health
  - version
  - port 3000
  - fingerprint
---

# Énumération

Grafana est facile à identifier et la version est décisive. Il écoute couramment sur le port 3000 (souvent mandaté en reverse proxy vers 80/443), et la page de connexion, l'endpoint non authentifié `/api/health`, et les ressources statiques versionnées divulguent toutes le build exact. Cette version importe car la traversée de chemin non authentifiée phare et d'autres vulnérabilités de plugins/serveur ne s'appliquent qu'à des plages spécifiques, de sorte que la lire d'abord vous indique si l'instance est exploitable sans identifiants et quels problèmes poursuivre.

```bash
# version without authentication
curl -sk https://<target>:3000/api/health            # {"database":"ok","version":"x.y.z",...}
curl -sk https://<target>:3000/login | grep -ioE 'Grafana v[0-9.]+'
```

## Notes d'exploitation

- `/api/health` renvoie la version sans authentification ; elle correspond directement à l'applicabilité de la [traversée de chemin](path-traversal-file-read.md) et des [exploits connus](known-exploits.md).
- La version décide de la présence de la lecture de fichier non authentifiée, ce qui change le besoin d'identifiants.
- Grafana est fréquemment exposé sur 3000 ou derrière un proxy ; scannez-le et lisez la version quelle que soit la façade.
- Orientez vers l'[authentification](authentication-and-anonymous-access.md), la traversée non authentifiée, ou les voies des sources de données selon l'accès et la version.

## Références

- [Grafana: HTTP API health](https://grafana.com/docs/grafana/latest/developers/http_api/other/)
- [Grafana security advisories](https://grafana.com/security/security-advisories/)
