---
order: 3
title: "RCE chains : du proxy front-end au back end privilégié"
description: "Les chaînes d'exécution de code à distance d'Exchange on-premises qui abusent du front end Client Access pour atteindre un back end privilégié (PowerShell, ECP, EWS) tournant en tant que SYSTEM : ProxyLogon, ProxyShell, ProxyNotShell et OWASSRF, et la désérialisation ViewState. Triage par build pour choisir la chaîne à portée."
keywords:
  - Exchange RCE
  - ProxyLogon
  - ProxyShell
  - ProxyNotShell
  - ViewState
---

# RCE chains

Une classe récurrente de failles Exchange partage une même forme. Le service **Client Access (front-end)** sert de proxy aux requêtes vers le **back end** et, dans ces chaînes, peut être trompé pour transmettre des requêtes **contrôlées par l'attaquant et implicitement de confiance** vers un endpoint back-end privilégié (le service PowerShell distant, ECP, ou EWS). Le back end fait confiance au front end et exécute la requête en tant que `SYSTEM` ou en tant qu'identité du pool d'applications Exchange. Les variations portent sur la façon dont le front end est dupé (un SSRF piloté par cookie, une confusion de chemin Autodiscover) et sur la primitive que le back end expose ensuite (un export de boîte aux lettres qui écrit un web shell, une commande PowerShell, un `__VIEWSTATE` signé qui se désérialise). Chaînez n'importe quelle ruse front-end à n'importe quelle écriture back-end et vous avez l'exécution de code.

## Triage : adapter le build à la chaîne

Le build que vous lisez lors de l'[énumération](../enumeration.md) décide quelle chaîne est à portée. Déclencher la mauvaise contre un build patché ne génère que des alertes.

```bash
# read the build, then match it to the in-range chain below
curl -sk -I https://<target>/owa/ | grep -i 'X-OWA-Version'   # e.g. 15.2.1118.x
# 15.0 = 2013, 15.1 = 2016, 15.2 = 2019; compare the build/rollup to the dates below
```

- **[ProxyLogon](proxylogon.md)**, pré-authentification. SSRF via le cookie `X-BEResource` atteint le `/ecp` back-end, s'authentifie en tant que n'importe quelle boîte aux lettres, puis une écriture dans le répertoire virtuel OAB dépose un web shell. À portée sur les builds au niveau ou en dessous du rollup du début 2021. Sans identifiant.
- **[ProxyShell](proxyshell.md)**, pré-authentification. Confusion de chemin Autodiscover atteint le back end PowerShell, qui exécute `New-MailboxExportRequest` pour écrire un shell `.aspx`. À portée sur les builds sous le rollup de mi-2021. Sans identifiant.
- **[ProxyNotShell](proxynotshell.md)**, authentifié avec un identifiant de boîte aux lettres valide. SSRF Autodiscover vers le back end PowerShell. À portée sous le rollup de fin 2022 ; la variante **OWASSRF** de la même page atteint le back end via le chemin `/owa/.../ecp` après la première mitigation de réécriture d'URL.
- **[Désérialisation ViewState](viewstate-deserialization.md)**, nécessite la `machineKey` ECP (statique, divulguée ou récupérée). Un `__VIEWSTATE` forgé signé avec ces clés se désérialise en exécution de code en tant que pool d'applications ECP. Indépendant du build ; dépend de l'exposition des clés.

## Pourquoi la récolte est si grande

Le back end s'exécute en tant que `SYSTEM` ou identité du pool d'applications Exchange, donc le shell est immédiatement à privilège élevé sur la machine. Parce qu'Exchange est privilégié dans AD (voir [PrivExchange](../privexchange.md)), `SYSTEM` sur un serveur Exchange est généralement à un pas de Domain Admin : dumpez LSASS et le compte machine, puis pivotez via les droits du compte machine Exchange ou le DCSync.

## Pages

- **[ProxyLogon](proxylogon.md)** : SSRF `X-BEResource` vers l'ECP back-end, puis une écriture OAB vers un web shell.
- **[ProxyShell](proxyshell.md)** : confusion de chemin Autodiscover vers le back end PowerShell, puis un export de boîte aux lettres vers un shell.
- **[ProxyNotShell](proxynotshell.md)** : SSRF Autodiscover authentifié vers PowerShell, avec les variantes OWASSRF et ApprovedApplicationCollection.
- **[Désérialisation ViewState](viewstate-deserialization.md)** : forger un `__VIEWSTATE` signé à partir d'une `machineKey` divulguée pour se désérialiser dans le pool d'applications ECP.

## Références

- [Google Cloud / Mandiant: ProxyShell exploiting Microsoft Exchange servers](https://cloud.google.com/blog/topics/threat-intelligence/pst-want-shell-proxyshell-exploiting-microsoft-exchange-servers)
- [Orange Tsai: ProxyLogon and the Exchange attack surface (Black Hat / DEVCORE)](https://devco.re/blog/2021/08/06/a-new-attack-surface-on-MS-exchange-part-1-ProxyLogon/)
- [Zero Day Initiative: exploiting Exchange PowerShell after ProxyNotShell](https://www.thezdi.com/blog/2024/9/11/exploiting-exchange-powershell-after-proxynotshell-part-2-approvedapplicationcollection)
