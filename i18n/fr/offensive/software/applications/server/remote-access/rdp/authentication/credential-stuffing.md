---
title: "Credential stuffing : rejouer des identifiants compromis contre RDP"
description: "Parce que RDP accepte les identifiants Windows et qu'il est largement exposé à internet, les attaquants rejouent des paires nom d'utilisateur/mot de passe issues de fuites et de compromissions antérieures contre lui. Contrairement au brute force, le stuffing essaie des paires connues comme valides, donc il est à faible volume et efficace, et c'est une technique dominante d'accès initial RDP précisément parce que les utilisateurs réutilisent les mots de passe entre services."
keywords:
  - credential stuffing
  - rdp
  - identifiants compromis
  - réutilisation de mot de passe
  - accès initial
---

# Credential stuffing

Le credential stuffing rejoue des paires nom d'utilisateur/mot de passe obtenues ailleurs, de fuites publiques, d'intrusions antérieures, de logs d'infostealer, contre un service dans l'espoir que les utilisateurs les ont réutilisées. RDP est une cible de choix : il est massivement exposé à internet, il accepte des identifiants Windows/de domaine ordinaires, et la réutilisation de mot de passe entre comptes personnels et professionnels est endémique. Le stuffing diffère du brute force en ce qu'il essaie un petit ensemble de paires connues comme valides plutôt que de deviner, de sorte qu'il reste sous les seuils de verrouillage et reste discret, ce qui explique précisément pourquoi c'est l'une des techniques dominantes d'accès initial RDP derrière les intrusions par rançongiciel.

```bash
# replay breach/stealer username:password pairs (1:1, not cross-product)
nxc rdp <target> -u leaked_users.txt -p leaked_pass.txt --no-bruteforce   # paired
hydra -C leaked_combos.txt rdp://<target> -t 1       # combo file user:pass
# source pairs from infostealer logs, breach compilations, and prior-compromise creds
```

## Notes d'exploitation

- Le stuffing utilise des paires connues, donc le volume est faible et le risque de verrouillage est minime ; alimentez-le avec des logs d'infostealer (qui incluent souvent l'hôte RDP et les identifiants exacts), des jeux de données de fuites et des identifiants récoltés plus tôt pendant la mission.
- Faites correspondre le format du nom d'utilisateur à la cible (local `host\user`, domaine `DOMAIN\user`, ou UPN) en utilisant le domaine appris lors de l'[énumération](../enumeration/banner-grabbing.md).
- Les logs d'infostealer sont particulièrement puissants pour RDP parce qu'ils capturent fréquemment l'identifiant RDP enregistré et l'adresse cible ensemble, faisant du « devinement » un rejeu direct.
- Un succès est un accès interactif avec les privilèges de cet utilisateur ; comme pour le brute force, attendez-vous à une réutilisation dans l'environnement et testez en conséquence.

## Références

- [CISA: RDP and credential-based intrusions](https://www.cisa.gov/news-events/cybersecurity-advisories)
- [HackTricks: RDP](https://book.hacktricks.xyz/network-services-pentesting/pentesting-rdp)
