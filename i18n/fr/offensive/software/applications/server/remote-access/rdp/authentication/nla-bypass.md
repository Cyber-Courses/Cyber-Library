---
title: "Contournement de NLA : considérations et mauvaises configurations de la Network Level Authentication"
description: "La Network Level Authentication oblige un client RDP à s'authentifier via CredSSP avant qu'une session ne soit créée, ce qui atténue l'exploitation pré-authentification et l'accès non authentifié à l'écran de connexion. L'angle offensif n'est pas de casser CredSSP mais d'exploiter les hôtes où NLA est désactivée ou non imposée, exposant la surface pré-authentification et l'écran de connexion, et d'enchaîner des identifiants volés à travers CredSSP."
keywords:
  - nla
  - credssp
  - network level authentication
  - surface pré-authentification
  - mauvaise configuration
---

# Contournement de NLA

La Network Level Authentication (NLA) exige qu'un client RDP s'authentifie via CredSSP avant que le serveur ne crée une session. C'est un durcissement significatif : il supprime l'accès non authentifié à l'écran de connexion et aux chemins de code pré-session où résidaient les bugs RDP vermiformes. L'angle offensif réaliste n'est pas de casser cryptographiquement CredSSP mais de cibler les endroits où NLA est absente ou non imposée. De nombreux hôtes, surtout les plus anciens ou tournés vers l'interne, tournent avec NLA désactivée, ce qui réexpose la surface pré-authentification et l'écran de connexion interactif aux clients non authentifiés ; et NLA n'arrête pas la devinette ou le rejeu d'identifiants, elle ne fait que déplacer la vérification dans CredSSP.

```bash
# determine whether NLA is required (the decisive fact)
nmap -p3389 --script rdp-enum-encryption <target>   # reports NLA supported vs required
# if NLA is NOT required, connect to the logon surface without credentials
xfreerdp /v:<target> /cert:ignore                   # reaches logon when NLA is off
# if NLA IS required, credentials are checked by CredSSP first -> use valid/stuffed creds
xfreerdp /v:<target> /u:'DOMAIN\user' /p:'pass' /cert:ignore
```

## Notes d'exploitation

- Le gain est de trouver NLA désactivée, pas de vaincre NLA : un hôte avec NLA désactivée expose la surface de protocole pré-authentification ([BlueKeep](../pre-authentication-flaws/bluekeep.md)/[DejaBlue](../pre-authentication-flaws/dejablue.md)) et l'écran de connexion aux connexions non authentifiées, ce que ne font pas les hôtes où NLA est exigée.
- Avec NLA exigée, les attaques se déplacent vers l'obtention d'identifiants valides (brute-force, spraying, stuffing) qui satisfont CredSSP ; NLA change l'endroit où les identifiants sont vérifiés, pas la possibilité de les deviner ou de les rejouer.
- CredSSP a historiquement connu ses propres vulnérabilités ; ce sont des bugs d'implémentation de protocole à rapprocher du niveau de correctif, distincts du cas de mauvaise configuration avec NLA désactivée.
- Énumérez d'abord l'état de NLA ([réglages de sécurité](../enumeration/security-settings.md)) ; il oriente toute l'attaque RDP.

## Références

- [Microsoft: Network Level Authentication and CredSSP](https://learn.microsoft.com/en-us/windows-server/remote/remote-desktop-services/)
- [HackTricks: RDP NLA](https://book.hacktricks.xyz/network-services-pentesting/pentesting-rdp)
