---
order: 4
title: "MITM et confiance : abus de la confiance SSH de clé d'hôte et de certificat"
description: "SSH authentifie le serveur par sa clé d'hôte, à laquelle on fait confiance à la première utilisation et qui est épinglée dans known_hosts, ou signée par une CA pour le SSH basé sur les certificats. Lorsque cette confiance est faible, utilisateurs qui ignorent les avertissements de clé d'hôte, clients configurés pour accepter n'importe quelle clé, ou failles de validation de certificat, un attaquant bien positionné réalise un machine-in-the-middle pour capturer les identifiants et détourner la session."
keywords:
  - mitm ssh
  - clé d'hôte
  - known_hosts
  - confiance à la première utilisation
  - certificat
---

# MITM et confiance

SSH ne résiste au machine-in-the-middle que parce que le client vérifie la clé d'hôte du serveur : à la première connexion, la clé est enregistrée dans `known_hosts` (trust-on-first-use), et les connexions ultérieures doivent correspondre ou le client avertit. Le SSH basé sur les certificats remplace cela par une CA qui signe les clés d'hôte. La surface d'attaque est là où cette confiance est faible : utilisateurs conditionnés à accepter les avertissements de changement de clé d'hôte, clients configurés pour accepter n'importe quelle clé (`StrictHostKeyChecking no`), nouvelles connexions sans clé encore épinglée, et failles de validation de certificat. Un attaquant bien positionné les exploite pour s'interposer, présentant sa propre clé d'hôte, déchiffrant la session, capturant le mot de passe ou la clé, et relayant vers le vrai serveur.

```bash
# position (ARP/DNS/route) + an SSH MITM proxy that presents a substitute host key
# the attack succeeds when the client does not reject the unknown/changed key
```

## Sous-thèmes

- **[Contournement de known_hosts](known-hosts-bypass.md)** : vaincre le trust-on-first-use et l'épinglage de clé.
- **[SSH MITM](ssh-mitm.md)** : s'interposer pour capturer les identifiants et détourner la session.
- **[Contournement de validation de certificat](certificate-validation-bypass.md)** : abus de la confiance SSH basée sur les certificats.

## Références

- [OpenSSH known_hosts and host keys](https://man.openbsd.org/sshd.8)
- [HackTricks: SSH MITM](https://book.hacktricks.xyz/network-services-pentesting/pentesting-ssh)
