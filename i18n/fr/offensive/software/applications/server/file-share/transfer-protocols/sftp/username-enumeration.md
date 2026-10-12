---
title: "Énumération des noms d'utilisateur : découvrir les comptes SFTP et SSH valides"
order: 2
description: "SSH peut révéler quels noms d'utilisateur sont valides via des différences de méthodes d'authentification et, sur les versions vulnérables, via des canaux auxiliaires temporels. Une liste d'utilisateurs validée concentre le spraying de mots de passe et les attaques par clé sur les comptes réels, et distingue les comptes limités à SFTP des comptes à shell complet, ce qui oriente la suite, du passage de l'accès au transfert à l'exécution de commandes."
keywords:
  - username enumeration
  - ssh
  - auth methods
  - timing
  - user list
---

# Énumération des noms d'utilisateur

Avant d'attaquer les identifiants, savoir quels noms d'utilisateur existent concentre l'effort et évite le bruit contre des comptes inexistants. SSH peut le révéler de plusieurs manières : les méthodes d'authentification annoncées diffèrent parfois entre utilisateurs valides et invalides, certaines versions d'OpenSSH présentaient un canal auxiliaire temporel où un utilisateur valide mettait un temps mesurablement plus long à être rejeté, et la bannière ou les méthodes autorisées peuvent distinguer les types de compte. Une liste validée pilote ensuite le spraying et les attaques par clé contre des cibles réelles et aide à séparer les comptes limités à SFTP de ceux à shell complet.

```bash
# which auth methods the server offers (and whether they vary per user)
ssh -o PreferredAuthentications=none -o PubkeyAuthentication=no user@<target>
nmap -p22 --script ssh-auth-methods --script-args="ssh.user=<user>" <target>
# timing-based enumeration on vulnerable OpenSSH (valid users slower to reject)
#   send a malformed/oversized auth and compare response timing across candidates
```

## Notes d'exploitation

- Le signal moderne le plus fiable est la différence de méthodes d'authentification acceptées par utilisateur (par exemple, un utilisateur configuré pour les clés uniquement par rapport à un mot de passe), que `ssh-auth-methods` fait ressortir ; le canal auxiliaire temporel ne s'applique qu'à des versions plus anciennes spécifiques d'OpenSSH.
- Amorcez les noms d'utilisateur candidats à partir d'autres énumérations (RID cycling SMB, comptes d'applications web, OSINT, noms de services courants) et validez-les ici avant de sprayer.
- Distinguer les comptes limités à SFTP (confinés au sous-système) des comptes à shell importe : les premiers mènent à l'[évasion du shell restreint](restricted-shell-and-chroot-escape.md), les seconds directement à l'exécution de commandes une fois authentifié.
- Restez peu bruyant ; l'énumération qui déclenche des échecs d'authentification est journalisée, préférez donc la vérification des différences de méthodes aux sondes temporelles massives lorsque c'est possible.

## Références

- [nmap ssh-auth-methods](https://nmap.org/nsedoc/scripts/ssh-auth-methods.html)
- [HackTricks : énumération d'utilisateurs SSH](https://book.hacktricks.xyz/network-services-pentesting/pentesting-ssh)
