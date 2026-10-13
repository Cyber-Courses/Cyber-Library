---
title: "Brute force de mot de passe : spraying et stuffing du portail VPN"
description: "Les portails SSL-VPN et l'IKE XAUTH acceptent un nom d'utilisateur et un mot de passe d'entreprise, ce qui en fait des cibles de choix pour le password spraying et le credential stuffing. Parce que les connexions VPN utilisent les mêmes informations d'identification d'annuaire que tout le reste, un mot de passe sprayé ou issu d'une fuite accorde fréquemment un tunnel vers le réseau interne, souvent là où le VPN constitue le périmètre."
keywords:
  - brute force vpn
  - password spray
  - credential stuffing
  - xauth
  - portail sslvpn
---

# Brute force de mot de passe

L'authentification VPN se valide habituellement contre l'annuaire d'entreprise, de sorte que le portail SSL-VPN et l'IKE XAUTH sont des cibles de grande valeur pour le password spraying et le credential stuffing : un seul mot de passe qui fonctionne est un tunnel vers le réseau interne, et comme le VPN constitue souvent le périmètre, c'est un accès profond. Le spraying d'un mot de passe courant sur de nombreux utilisateurs évite le verrouillage et trouve les comptes faibles ; le stuffing rejoue des informations d'identification issues de fuites et d'infostealers, qui aboutissent parce que les connexions VPN réutilisent les mêmes mots de passe que tout le reste. La MFA, lorsqu'elle est appliquée, est la barrière restante (et est parfois contournable ou non appliquée à toutes les voies d'authentification).

```bash
# SSL-VPN portal spraying (product-specific login endpoint)
# generic HTTP-POST form spray with a lockout-safe single password across users:
hydra -L users.txt -p 'Spring2025!' <target> https-post-form \
  "/remote/logincheck:username=^USER^&credential=^PASS^:<fail-string>"
# IKE XAUTH credential attack (after PSK/main-mode)
# learn the lockout policy and spray below it; stuff breached pairs 1:1
```

## Notes d'exploitation

- Sprayez un mot de passe sur des utilisateurs validés pour rester sous le seuil de verrouillage, et faites du stuffing des couples issus de fuites/infostealers séparément ; les portails VPN sont une cible de stuffing de premier ordre parce qu'ils acceptent les informations d'identification d'entreprise réutilisées.
- Dérivez les noms d'utilisateur et la politique de mot de passe d'autres énumérations (l'annuaire, l'OSINT, les différences de messages d'erreur du portail) ; faites correspondre le format de nom d'utilisateur attendu par le portail (UPN, `DOMAIN\user`, ou nom nu).
- Le point de terminaison de connexion et le signal d'échec sont spécifiques au produit (FortiGate `/remote/logincheck`, d'autres diffèrent) ; identifiez d'abord l'appliance, voir [récupération de bannière](../enumeration/banner-grabbing.md).
- Une information d'identification valide sans MFA est un tunnel complet ; là où la MFA est appliquée, notez les voies d'authentification qui peuvent la sauter (IKE XAUTH hérité, portails spécifiques) et les [exploits d'appliance](../ssl-vpn-appliances/index.md) qui contournent entièrement l'authentification.

## Outils

- [hydra](https://github.com/vanhauser-thc/thc-hydra)
- [NetExec](https://github.com/Pennyw0rth/NetExec)

## Références

- [CISA: VPN password spraying](https://www.cisa.gov/news-events/cybersecurity-advisories)
- [HackTricks: VPN](https://book.hacktricks.xyz/network-services-pentesting/ipsec-ike-vpn-pentesting)
