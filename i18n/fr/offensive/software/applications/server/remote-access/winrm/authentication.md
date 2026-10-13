---
order: 2
title: "Authentification : mots de passe WinRM, NTLM et pass-the-hash"
description: "WinRM s'authentifie avec des identifiants Windows via Negotiate/NTLM, Kerberos ou Basic. Parce qu'il accepte NTLM, un attaquant s'authentifie avec un hash NT (pass-the-hash) aussi bien qu'avec un mot de passe, et un ticket Kerberos fonctionne également. Le spraying et la réutilisation d'identifiants contre WinRM sont une étape standard de mouvement latéral ; seuls les comptes autorisés peuvent se connecter."
keywords:
  - winrm authentication
  - pass-the-hash
  - ntlm
  - kerberos
  - spray
---

# Authentification

WinRM s'authentifie avec des identifiants Windows via plusieurs mécanismes : Negotiate (qui sélectionne Kerberos ou NTLM), NTLM brut, Kerberos, Basic (mot de passe dans un en-tête HTTP, utilisé sur HTTPS) et CredSSP. La conséquence importante est que WinRM accepte NTLM, donc un attaquant s'authentifie avec un hash NT via pass-the-hash exactement comme avec un mot de passe, et un ticket Kerberos (issu d'un compromis antérieur) fonctionne via Negotiate. L'attaque pratique est donc la réutilisation d'identifiants et de hashes : spraying de mots de passe, rejeu de hashes NT capturés ou extraits, ou présentation d'un ticket Kerberos, contre WinRM pour le mouvement latéral. La contrainte est l'autorisation : seuls les administrateurs locaux et les Remote Management Users peuvent se connecter, donc un identifiant valide n'est pas toujours un identifiant utilisable.

```bash
# password, NT hash (pass-the-hash), and ticket all work
nxc winrm <target> -u user -p 'Password1'
nxc winrm <target> -u user -H <nthash>                 # pass-the-hash
evil-winrm -i <target> -u user -H <nthash>             # hash auth into a shell
evil-winrm -i <target> -u user -p 'Password1'
# Kerberos: export a ticket and use Negotiate
export KRB5CCNAME=ticket.ccache; evil-winrm -i <target> -u user -r <REALM>
```

## Notes d'exploitation

- Le pass-the-hash est l'essentiel : la prise en charge de NTLM par WinRM signifie qu'un hash NT extrait s'authentifie sans cassage, de sorte que WinRM est une destination privilégiée pour les hashes récoltés ailleurs.
- L'autorisation conditionne l'usage : le compte doit être administrateur local ou membre de Remote Management Users sur la cible ; `nxc winrm` distingue un compte valide-mais-non-autorisé d'un compte utilisable.
- Pratiquez le spraying et la réutilisation comme avec n'importe quel service Windows, en respectant le verrouillage ; un succès que `nxc` marque `Pwn3d!` est admin via WinRM et donne un shell privilégié immédiat.
- L'authentification Kerberos évite entièrement NTLM et convient aux environnements qui journalisent/limitent NTLM ; présentez un TGT/ticket de service via Negotiate.

## Références

- [HackTricks: WinRM authentication](https://book.hacktricks.xyz/network-services-pentesting/5985-5986-pentesting-winrm)
- [Evil-WinRM](https://github.com/Hackplayers/evil-winrm)
