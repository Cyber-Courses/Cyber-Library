---
title: "Injection de commandes : RCE authentifiée dans LibreNMS et Observium"
description: "LibreNMS et Observium construisent des commandes shell à partir d'entrées fournies par l'utilisateur dans plusieurs fonctionnalités (ajout d'équipement, options SNMP, paramètres de services et collectd), et là où cette entrée n'est pas assainie, un attaquant authentifié injecte des commandes qui s'exécutent sur l'hôte de supervision en tant qu'utilisateur web ou poller. Faire correspondre la version à la fonctionnalité affectée donne l'exécution de code."
keywords:
  - injection de commandes LibreNMS
  - observium
  - rce
  - options SNMP
  - poller
---

# Injection de commandes

Les deux plateformes appellent des outils externes (binaires net-snmp, `fping`, `rrdtool`, scripts de collecteurs) construits à partir de valeurs de configuration, et plusieurs de ces valeurs sont contrôlées par l'attaquant via l'UI/API authentifiée. Là où l'entrée est placée dans une commande sans assainissement, injecter des métacaractères shell exécute des commandes arbitraires sur l'hôte de supervision. LibreNMS a eu des injections de commandes via les paramètres d'équipement et de service, les champs d'options SNMP, et les entrées collectd/graphe ; Observium via une gestion similaire des équipements et des options. La commande injectée s'exécute en tant qu'utilisateur du service web ou poller. La fonctionnalité précise affectée et le fait qu'elle requière un privilège admin ou inférieur sont spécifiques à la version, donc identifier et faire correspondre l'avis sélectionne le chemin.

```bash
# the injectable field is version/feature-specific; the pattern is a config value
# that reaches a shell command. Examples (authenticated):
#   device "overwrite_ip" / SNMP options field:  127.0.0.1 -c public; id
#   service check parameters / collectd inputs carrying `;`, `|`, $(...)
# the injected command runs on the monitoring host (web/poller user). Match the
# LibreNMS/Observium version to the advisory for the exact field.
curl -sk -b cj 'https://<target>/ajax_form.php' --data 'type=...&snmp_options=127.0.0.1;id'
```

## Notes d'exploitation

- Les entrées injectables sont des valeurs de configuration qui aboutissent à des commandes shell : chaînes d'options SNMP, adresses/champs de réécriture d'équipement, et paramètres de service/collecteur ; testez-les avec des métacaractères une fois authentifié.
- L'exécution se fait en tant qu'utilisateur web/poller sur l'hôte de supervision, qui a accès à la base de données détenant les [identifiants stockés](credential-harvesting.md) de chaque équipement, donc la RCE se propage au parc.
- Le champ exact et le privilège requis sont spécifiques à la version ; identifiez ([énumération](enumeration.md)) et faites correspondre l'avis.
- Combinez avec l'[authentification](authentication.md) pour l'étape d'accès ; à partir de l'exécution, récoltez les identifiants stockés et pivotez.

## Références

- [LibreNMS security advisories](https://github.com/librenms/librenms/security/advisories)
- [HackTricks](https://book.hacktricks.xyz/)
