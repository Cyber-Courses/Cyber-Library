---
title: "NFSv4 et Kerberos : la configuration plus robuste et ses points faibles"
order: 4
description: "NFSv4 ajoute un unique pseudo-système de fichiers, un mappage d'identité basé sur les noms et une sécurité Kerberos optionnelle (sec=krb5). Kerberos authentifie l'utilisateur et, avec krb5i/krb5p, protège l'intégrité et la confidentialité, déjouant l'usurpation d'UID. Les points faibles sont les serveurs qui autorisent encore le repli sur sec=sys, un mappage d'identité mal configuré, et la dépendance à des tickets ou keytabs Kerberos volés."
keywords:
  - nfsv4
  - sec=krb5
  - idmapd
  - keytab
  - kerberos
---

# NFSv4 et Kerberos

NFSv4 modernise le protocole : un unique pseudo-système de fichiers exporté au lieu de montages par export, un mappage d'identité basé sur les noms (`user@domain` résolu par `idmapd`) plutôt que des UID numériques bruts, et une sécurité Kerberos optionnelle. Avec `sec=krb5`, l'utilisateur est authentifié par un ticket Kerberos, et `krb5i` et `krb5p` ajoutent une protection d'intégrité et de confidentialité, ce qui déjoue les attaques d'usurpation d'UID d'`AUTH_SYS` parce que l'identité est désormais établie cryptographiquement plutôt qu'affirmée. L'intérêt offensif se porte donc sur les configurations qui n'atteignent pas ce niveau et sur l'abus des identifiants Kerberos plutôt que la forge d'UID.

## Points faibles

```bash
# does the server still allow sec=sys (AUTH_SYS) alongside or instead of krb5?
mount -t nfs -o vers=4,sec=sys <target>:/ /mnt/nfs   # if this succeeds, spoofing applies
# what security flavours are offered?
nmap -p2049 --script nfs-showmount <target>; rpcinfo -p <target>
```

- **Repli sur sec=sys** : de nombreux déploiements activent Kerberos mais acceptent encore `sec=sys`, de sorte qu'un attaquant monte simplement avec `sec=sys` et les attaques d'[usurpation d'UID/GID](uid-and-gid-spoofing.md) s'appliquent inchangées. C'est la faille la plus courante dans le monde réel.
- **Mauvaise configuration du mappage d'identité** : si les domaines ou mappages d'`idmapd` sont incohérents, les identités peuvent se réduire à `nobody` ou se mapper de façon inattendue, élargissant parfois l'accès.
- **Identifiants Kerberos volés** : là où `krb5` est imposé, l'accès nécessite un ticket valide ; l'attaque se déplace vers l'obtention du TGT/ticket de service d'un utilisateur ou d'un keytab d'hôte (`/etc/krb5.keytab`) et son utilisation pour monter en tant que ce principal.

```bash
# with a stolen keytab or ticket, authenticate then mount as that principal
kinit -kt /loot/krb5.keytab host/server@REALM
mount -t nfs -o vers=4,sec=krb5 <target>:/ /mnt/nfs   # access as the authenticated identity
```

## Notes d'exploitation

- Testez toujours `sec=sys` d'abord : un serveur à Kerberos imposé qui permet encore le repli sur AUTH_SYS est entièrement exposé à l'usurpation d'UID, ce qui est une mauvaise configuration fréquente.
- Sous `krb5` imposé, l'attaque est le vol d'identifiants, pas la forge : un keytab capturé authentifie en tant que principal d'hôte ou de service ; un TGT volé (depuis un client compromis) authentifie en tant que l'utilisateur.
- `krb5p` chiffre le trafic, donc la capture passive des données NFS est déjouée ; `krb5` (authentification seule) laisse encore les données lisibles sur le réseau.
- Le domaine de mappage d'identité doit correspondre entre le client et le serveur pour que les noms se résolvent ; les incohérences sont une source à la fois de refus et, occasionnellement, de mappage trop large.

## Références

- [RFC 8881: NFSv4.1 security](https://datatracker.ietf.org/doc/html/rfc8881)
- [nfs(5): sec= options](https://man7.org/linux/man-pages/man5/nfs.5.html)
- [Linux idmapd](https://man7.org/linux/man-pages/man8/rpc.idmapd.8.html)
