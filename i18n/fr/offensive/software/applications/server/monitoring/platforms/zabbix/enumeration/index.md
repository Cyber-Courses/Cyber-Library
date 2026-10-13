---
title: "Énumération : cartographier Zabbix via l'API et l'interface"
order: 3
description: "Zabbix expose une API JSON-RPC et une interface web qui révèlent la version (souvent sans authentification), les comptes utilisateurs et les hôtes supervisés. La version guide le choix des exploits, la liste des utilisateurs guide les attaques sur les identifiants, et l'inventaire des hôtes supervisés est une carte du parc interne avec les IP et services que Zabbix atteint, le tout obtenu via api_jsonrpc.php une fois authentifié ou, pour la version, avant."
keywords:
  - zabbix enumeration
  - api_jsonrpc.php
  - apiinfo.version
  - host.get
  - user.get
---

# Énumération

L'énumération de Zabbix se fait via l'API JSON-RPC (`/api_jsonrpc.php`) et l'interface web. La version est lisible, généralement sans authentification, via `apiinfo.version` et le pied de page de la page de connexion, et c'est le fait clé pour le choix des exploits. Après authentification (ou via n'importe quelle session), l'API énumère les comptes utilisateurs (`user.get`), ce qui alimente les attaques sur les identifiants, et les hôtes supervisés (`host.get`), qui constituent une carte du parc interne : les adresses IP, noms d'hôtes et interfaces que Zabbix atteint, ainsi que les items et macros qui portent souvent les identifiants utilisés pour les superviser. L'API est la voie efficace ; l'interface affiche les mêmes données.

```bash
Z=https://<target>/zabbix/api_jsonrpc.php; H='Content-Type: application/json-rpc'
# version (no auth in many versions)
curl -sk $Z -H "$H" -d '{"jsonrpc":"2.0","method":"apiinfo.version","params":{},"id":1}'
# authenticate to get a token, then enumerate
TOK=$(curl -sk $Z -H "$H" -d '{"jsonrpc":"2.0","method":"user.login","params":{"username":"Admin","password":"zabbix"},"id":1}' | jq -r .result)
curl -sk $Z -H "$H" -d '{"jsonrpc":"2.0","method":"user.get","params":{"output":["userid","username","roleid"]},"auth":"'"$TOK"'","id":1}'
curl -sk $Z -H "$H" -d '{"jsonrpc":"2.0","method":"host.get","params":{"output":["host"],"selectInterfaces":["ip","dns"]},"auth":"'"$TOK"'","id":1}'
```

## Sous-thèmes

- **[Détection de version](version-detection.md)** : lire la version de Zabbix.
- **[API](api.md)** : l'API JSON-RPC comme interface d'énumération.
- **[Utilisateur](user.md)** : énumérer les comptes.
- **[Hôte](host.md)** : l'inventaire des hôtes supervisés.

## Références

- [Zabbix API : apiinfo.version](https://www.zabbix.com/documentation/current/en/manual/api/reference/apiinfo/version)
- [HackTricks : Zabbix](https://book.hacktricks.xyz/network-services-pentesting/pentesting-zabbix)
