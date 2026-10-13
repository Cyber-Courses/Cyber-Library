---
title: "Brute force de mot de passe : devinement en ligne contre la connexion RDP"
description: "RDP accepte les identifiants Windows, donc le brute force et le password spraying ciblent directement la connexion. Le verrouillage de compte Windows et le coût de chaque handshake RDP font du spraying low-and-slow d'utilisateurs validés l'approche pratique ; un succès donne un bureau interactif, souvent administratif, et il est généralement réutilisable sur le domaine."
keywords:
  - brute force rdp
  - password spray
  - verrouillage de compte
  - crowbar
  - nxc rdp
---

# Brute force de mot de passe

RDP s'authentifie contre les comptes Windows, donc le devinement en ligne va droit sur la connexion. Deux contraintes le façonnent : la politique de verrouillage de compte Windows (quelques tentatives erronées peuvent verrouiller un compte de domaine, causant du bruit et un déni) et le coût relatif de chaque connexion RDP. La méthode efficace est donc le password spraying, un mot de passe soigneusement choisi sur de nombreux noms d'utilisateur validés, restant sous le seuil de verrouillage, plutôt qu'un brute force profond par compte. Un devinement réussi renvoie un bureau interactif, couramment avec des droits administratifs, et l'identifiant est généralement réutilisable sur le domaine.

```bash
# spray one password across users, respecting lockout (single-threaded, paced)
nxc rdp <target> -u users.txt -p 'Spring2025!'         # flags valid creds and admin access
crowbar -b rdp -s <target>/32 -u admin -C passwords.txt -n 1
hydra -L users.txt -p 'Spring2025!' rdp://<target> -t 1
# derive the lockout threshold first (from SMB/LDAP) to spray safely
nxc smb <dc> -u user -p pass --pass-pol 2>/dev/null
```

## Notes d'exploitation

- Apprenez la politique de verrouillage avant de sprayer (via SMB/LDAP `--pass-pol`) : sprayez moins de tentatives que le seuil par fenêtre, et attendez la réinitialisation, sinon vous risquez de verrouiller des comptes et d'alerter les défenseurs.
- Sprayez des noms d'utilisateur validés (à partir des données de domaine des [infos NTLM RDP](../enumeration/banner-grabbing.md) plus l'énumération AD) sous forme `domain\user` ou UPN ; un seul mot de passe courant sur l'organisation trouve les comptes faibles.
- Le module RDP de NetExec signale non seulement la validité mais aussi si le compte a un accès admin sur la cible, vous dirigeant vers les succès de plus grande valeur.
- Un identifiant RDP valide est un accès interactif et un bon matériau de réutilisation ; testez-le sur d'autres hôtes et services, et attendez-vous à ce que de nombreux comptes RDP soient admins locaux.

## Outils

- [NetExec (rdp)](https://github.com/Pennyw0rth/NetExec)
- [crowbar](https://github.com/galkan/crowbar)

## Références

- [HackTricks: RDP brute force](https://book.hacktricks.xyz/network-services-pentesting/pentesting-rdp)
- [Microsoft: account lockout policy](https://learn.microsoft.com/en-us/windows/security/threat-protection/security-policy-settings/account-lockout-policy)
