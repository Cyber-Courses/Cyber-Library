---
title: "Authentification : attaquer la connexion RDP"
description: "RDP s'authentifie avec des identifiants Windows, c'est donc une cible pour le brute-force, le password-spray et le credential-stuffing, et un point d'atterrissage de choix pour les mots de passe réutilisés et compromis. La Network Level Authentication change l'endroit où les identifiants sont vérifiés mais pas la possibilité de les deviner. Des identifiants RDP valides donnent un bureau interactif, souvent avec des privilèges locaux ou de domaine."
keywords:
  - authentification rdp
  - brute force
  - password spray
  - credential stuffing
  - nla
---

# Authentification

RDP se connecte avec des identifiants Windows (locaux ou de domaine), ce qui en fait une cible directe des attaques par devinette et par réutilisation et l'un des moyens les plus courants par lesquels des identifiants volés ou pulvérisés se transforment en accès interactif. Le brute-force et le spraying fonctionnent contre la connexion, le credential-stuffing rejoue des paires nom d'utilisateur/mot de passe compromises, et la Network Level Authentication (NLA) ne fait que déplacer la vérification plus tôt (dans CredSSP) plutôt que d'empêcher la devinette. Un identifiant valide donne un bureau interactif complet, fréquemment avec des droits administratifs ou de domaine, de sorte que l'authentification RDP est une surface d'attaque à forte valeur et à forte fréquence, soumise au verrouillage de compte Windows.

```bash
# spray/guess against RDP, respecting lockout (domain\user forms from enumeration)
nxc rdp <target> -u users.txt -p 'Winter2025!'          # marks valid + whether admin
hydra -L users.txt -p 'Winter2025!' rdp://<target> -t 1
```

## Sous-thèmes

- **[Brute-force de mot de passe](password-brute-force.md)** : devinette en ligne contre la connexion.
- **[Credential stuffing](credential-stuffing.md)** : rejeu d'identifiants compromis et réutilisés.
- **[Contournement de NLA](nla-bypass.md)** : considérations et mauvaises configurations de la Network Level Authentication.

## Références

- [HackTricks: RDP authentication](https://book.hacktricks.xyz/network-services-pentesting/pentesting-rdp)
- [Microsoft: RDP security](https://learn.microsoft.com/en-us/windows-server/remote/remote-desktop-services/)
