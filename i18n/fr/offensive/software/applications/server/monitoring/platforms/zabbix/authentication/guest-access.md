---
title: "Accès invité : le compte Zabbix en lecture seule intégré"
order: 3
description: "Zabbix dispose d'un compte invité intégré qui, lorsqu'il est activé, accorde un accès en lecture non authentifié à tout ce que le groupe d'utilisateurs invité est autorisé à voir, couramment les tableaux de bord et les données d'hôte. Historiquement activé par défaut, l'accès invité expose l'inventaire des hôtes supervisés et la configuration sans aucun identifiant, et des permissions invité trop larges peuvent étendre cela à des données sensibles."
keywords:
  - guest access
  - zabbix guest
  - read-only
  - unauthenticated
  - dashboards
---

# Accès invité

Zabbix inclut un utilisateur `guest` spécial qui fournit un accès aux visiteurs non authentifiés : lorsqu'il est activé, quiconque atteint le frontend est traité comme l'utilisateur invité et voit tout ce que le groupe d'utilisateurs de l'invité est autorisé à voir. Historiquement, l'accès invité était activé par défaut, et il est toujours présent là où il n'a pas été explicitement désactivé. L'exposition dépend des permissions du groupe invité : au minimum, il révèle généralement les tableaux de bord et la liste des hôtes supervisés, soit une carte du parc interne ; là où un administrateur a accordé au groupe invité des permissions de lecture plus larges (une commodité courante), il expose les données des éléments, les problèmes et la configuration. Tout cela sans aucun identifiant.

```bash
# simply browsing the frontend uses the guest session if enabled
curl -sk https://<target>/zabbix/zabbix.php?action=dashboard.view | grep -i dashboard
# the API confirms guest login (empty password) where enabled
curl -sk https://<target>/zabbix/api_jsonrpc.php -H 'Content-Type: application/json-rpc' \
  -d '{"jsonrpc":"2.0","method":"user.login","params":{"username":"guest","password":""},"id":1}'
# then enumerate what guest can see (hosts, dashboards)
```

## Notes d'exploitation

- L'accès invité est non authentifié : atteindre le frontend est la seule exigence, de sorte qu'un Zabbix exposé avec l'invité activé divulgue des données à quiconque.
- La valeur augmente avec les permissions du groupe invité : un invité par défaut voit l'inventaire des hôtes (carte du parc), tandis qu'un groupe invité trop permissif expose les valeurs des éléments, les problèmes et la configuration, qui peuvent contenir des détails sensibles.
- Il est en lecture seule, de sorte qu'il ne fournit pas à lui seul d'exécution de code, mais les données d'hôte/macro qu'il expose constituent de la reconnaissance et parfois des identifiants ; combinez-le avec les pages d'énumération.
- Vérifiez la [version](../enumeration/version-detection.md) : l'invité a été désactivé par défaut dans les versions ultérieures, de sorte que sa présence indique une instance plus ancienne ou configurée de manière permissive.

## Références

- [Zabbix : utilisateur invité et permissions](https://www.zabbix.com/documentation/current/en/manual/web_interface/frontend_sections/users/user_groups)
- [HackTricks : Zabbix](https://book.hacktricks.xyz/network-services-pentesting/pentesting-zabbix)
