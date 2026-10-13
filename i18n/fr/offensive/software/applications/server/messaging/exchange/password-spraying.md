---
order: 2
title: "Password spraying : identifiants de domaine au périmètre"
description: "Sprayer des mots de passe contre les endpoints Exchange exposés sur Internet (OWA, EWS, Autodiscover, ActiveSync) pour transformer une liste d'utilisateurs en identifiant de domaine depuis l'extérieur, lire 200 contre 401 sur EWS, mener une cadence consciente du verrouillage, et exploiter les endpoints qui sautent le suivi du verrouillage."
keywords:
  - Exchange password spraying
  - OWA
  - EWS
  - ActiveSync
  - Basic auth
---

# Password spraying

Les endpoints web d'Exchange s'authentifient face à **Active Directory**, donc un mot de passe qui fonctionne contre OWA ou EWS est un mot de passe de domaine. Parce que les endpoints sont exposés sur Internet, les sprayer est la façon standard de convertir une [liste d'utilisateurs](enumeration.md) récoltée en un **foothold de domaine depuis l'extérieur**, sans VPN ni accès interne. La mécanique compte : différents endpoints appliquent le verrouillage différemment, et certains rapportent un identifiant valide plus proprement que d'autres.

## Quel endpoint, et à quoi ressemble un coup

Quatre endpoints s'appuient tous sur la même authentification AD, et chacun est sprayable en Basic ou NTLM :

- **OWA** (`/owa/auth.owa`) : un POST de formulaire ; un identifiant valide redirige dans la boîte aux lettres, un mauvais ressert le formulaire de connexion.
- **EWS** (`/ews/exchange.asmx`) : HTTP Basic/NTLM ; un identifiant valide renvoie **200** (ou un fault SOAP à propos du corps de la requête), un mauvais renvoie **401**. C'est le signal le plus propre.
- **Autodiscover** (`/autodiscover/autodiscover.xml`) : HTTP Basic/NTLM, même comportement 200-contre-401.
- **ActiveSync** (`/Microsoft-Server-ActiveSync`) : HTTP Basic ; fréquemment **pas** suivi par les mêmes compteurs de verrouillage que l'organisation surveille sur OWA.

Une sonde EWS brute rend le signal explicite :

```bash
# EWS with Basic auth: 401 = bad credential, 200/500 = the credential authenticated
curl -sk -o /dev/null -w '%{http_code}\n' -u 'EXAMPLE\john.doe:Autumn2026!' \
  'https://mail.example.com/ews/exchange.asmx'
# 200  -> valid, reached the service
# 401  -> rejected credential
# loop a user list at one password, keep every non-401
```

Lire le code est tout le jeu : `401` est un identifiant rejeté, tout le reste (200, ou un fault SOAP 500 se plaignant de l'absence de corps de requête) signifie que l'identifiant a passé l'authentification et que seul le contenu de la requête était erroné.

## Sprayer avec MailSniper

```powershell
# One password across the whole user list, against OWA then EWS
Invoke-PasswordSprayOWA -ExchHostname mail.example.com -UserList gal.txt -Password 'Spring2026!' -OutFile hits.txt
Invoke-PasswordSprayEWS -ExchHostname mail.example.com -UserList gal.txt -Password 'Spring2026!' -OutFile hits.txt
# ActiveSync, which often bypasses OWA lockout tracking
Invoke-PasswordSprayEAS -ExchHostname mail.example.com -UserList gal.txt -Password 'Spring2026!'
```

`hits.txt` contient les paires `user:password` qui se sont authentifiées. Une ligne est un identifiant de domaine : alimentez-la directement dans le [dump de la GAL](enumeration.md), l'énumération AD authentifiée, le [Kerberoasting](../../directory/active-directory/authentication/kerberos/roasting.md), et [ProxyNotShell](rce-chains/proxynotshell.md), qui nécessite exactement un identifiant valide.

## Cadence consciente du verrouillage

La même discipline que le [password spraying](../../directory/active-directory/authentication/credentials/password-spraying.md) de domaine s'applique, avec des spécificités Exchange :

- Sprayez **un mot de passe par fenêtre de verrouillage** sur tous les utilisateurs, jamais plusieurs mots de passe par utilisateur. Lisez d'abord la politique : authentifié, `Get-ADDefaultDomainPasswordPolicy` ou `nxc ... --pass-pol` ; non authentifié, supposez le courant de 5 tentatives par 30 minutes et laissez de la marge.
- **Comptez les tentatives sur chaque endpoint.** EWS, OWA et Autodiscover incrémentent tous le même `badPwdCount` sur le compte AD, donc trois sprays sur trois endpoints sont trois échecs contre le même compteur. ActiveSync est l'exception fréquente qui ne nourrit pas le compteur que l'organisation surveille, raison pour laquelle c'est l'endpoint de spray le plus sûr et celui à utiliser quand le seuil de verrouillage est serré.
- Rythmez-le. Le spraying exposé sur Internet est journalisé par le serveur et généralement par un WAF ou un proxy d'identité, donc commencez par un seul mot de passe à forte probabilité (saison plus année, nom de l'entreprise plus année) plutôt qu'un dictionnaire.

## Notes d'exploitation

- Un seul coup OWA/EWS est un **identifiant de domaine**, pas seulement un accès au courrier : il déverrouille l'énumération LDAP, le roasting, et le chemin RCE à identifiant unique en une étape.
- Les endpoints hérités (EWS, ActiveSync, Autodiscover en Basic) échappent souvent au MFA que l'organisation croit appliquer sur OWA, donc ce sont les cibles les plus molles même dans un parc nominalement protégé par MFA.
- Le [dump de la GAL](enumeration.md) et le spray se renforcent mutuellement : un identifiant dumpe la vraie liste d'utilisateurs, ce qui fait atterrir proprement le spray du mot de passe suivant au lieu de verrouiller des comptes sur des noms invalides.
- Le 200-contre-401 d'EWS est le signal à automatiser ; la redirection de formulaire d'OWA est plus bruyante et plus lente à analyser, donc préférez EWS ou Autodiscover pour le spray brut et gardez OWA pour la confirmation.

## Outils

- **MailSniper** (`Invoke-PasswordSprayOWA` / `-EWS` / `Invoke-PasswordSprayEAS`) : spraying d'endpoint avec journalisation des coups.
- **TREVORspray** (Black Lantern Security) : spraying distribué, conscient du verrouillage, à travers des proxies.
- **o365spray / ruler** : sprayers alternatifs pour Exchange et les endpoints hybrides.

## Références

- [MailSniper (dafthack)](https://github.com/dafthack/MailSniper)
- [pentestlab: Microsoft Exchange password spraying](https://pentestlab.blog/2019/09/05/microsoft-exchange-password-spraying/)
- [TREVORspray (Black Lantern Security)](https://github.com/blacklanternsecurity/TREVORspray)
