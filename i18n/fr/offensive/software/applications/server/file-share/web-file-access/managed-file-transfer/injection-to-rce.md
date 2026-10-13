---
title: "Injection vers RCE : escalader les failles MFT vers l'exécution de code"
description: "Les appliances MFT ont porté des failles d'injection qui atteignent l'exécution de code à distance : injection SQL utilisée pour écrire des fichiers ou appeler des procédures de base de données, désérialisation non sûre d'objets contrôlés par l'attaquant, et injection de commande dans des fonctionnalités qui invoquent l'OS. Chaînées depuis un contournement d'authentification ou atteignables sans authentification, elles donnent l'exécution de code sur l'appliance et les données qu'elle détient."
keywords:
  - injection sql
  - désérialisation
  - injection de commande
  - mft
  - exécution de code à distance
---

# Injection vers RCE

Les vulnérabilités MFT qui ont causé le plus de dégâts ont escaladé d'une faille d'injection vers l'exécution de code à distance sur l'appliance. Trois classes reviennent. L'injection SQL, atteinte sans authentification ou via un contournement, est utilisée non seulement pour lire des données mais pour écrire des fichiers dans la webroot ou invoquer une fonctionnalité base-de-données-vers-OS, transformant une faille de requête en exécution. La désérialisation non sûre d'objets fournis par l'attaquant (courante dans les backends MFT Java et .NET) exécute du code lors de la reconstruction de l'objet. Et l'injection de commande dans des fonctionnalités qui invoquent un shell (gestion d'archives, scripting, intégrations) exécute des commandes OS directement. L'appliance détient les fichiers de chaque partenaire, de sorte que l'exécution de code à cet endroit est à la fois une compromission du système et un vol de données de masse.

```bash
# the shape varies by product; the pattern is inject -> write/execute
# SQLi to file write (e.g. stacked query writing a web shell to the served path)
curl -sk "https://<target>/<endpoint>?id=1;<stacked-insert-writing-aspx>"
# deserialization: send a crafted serialized object to the vulnerable parameter
curl -sk -X POST https://<target>/<endpoint> --data-binary @gadget.bin -H 'Content-Type: application/octet-stream'
# command injection in a feature that invokes the OS
curl -sk -X POST https://<target>/<endpoint> -d 'field=valid; id'
```

## Notes d'exploitation

- L'escalade, et non l'injection seule, est l'objectif : une SQLi qui écrit un web shell dans le répertoire servi, une chaîne de gadgets de désérialisation pour le framework backend, ou un point d'injection de commande dans une fonctionnalité qui invoque l'OS.
- Beaucoup de celles-ci chaînent depuis un [contournement d'authentification](authentication-bypass.md) (l'endpoint vulnérable se trouve derrière une authentification elle-même contournable), tandis que certaines sont atteignables sans authentification d'emblée.
- L'exécution de code atterrit sur l'appliance sous le compte de service (souvent fortement privilégié), exposant la configuration, les clés de chiffrement et chaque fichier transitant ; priorisez les données stockées et les clés.
- Celles-ci sont spécifiques au produit et à la version ; les pages des [exploits MFT connus](known-mft-exploits/index.md) décrivent les campagnes réelles et quelle classe chacune a utilisée.

## Références

- [OWASP: injection](https://owasp.org/www-community/Injection_Flaws)
- [CISA: MFT exploitation advisories](https://www.cisa.gov/news-events/cybersecurity-advisories)
