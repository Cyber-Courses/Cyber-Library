---
title: "Contournement d'authentification : forger et abuser des jetons d'authentification Zimbra"
order: 2
description: "Obtenir un jeton Zimbra valide sans le mot de passe du compte : usurpation admin-vers-utilisateur via le SOAP DelegateAuthRequest avec un jeton admin, le mécanisme preauth et sa clé de preauth de domaine, et les failles de traitement des jetons qui renvoient un ZM_AUTH_TOKEN ou ZM_ADMIN_AUTH_TOKEN utilisable à partir d'une requête forgée. Enveloppes SOAP guidées avec interprétation des réponses."
keywords:
  - zimbra authentication bypass
  - delegateauthrequest
  - zm_admin_auth_token
  - zimbra preauth
  - admin impersonation
---

# Contournement d'authentification

Tout le modèle d'autorisation de Zimbra repose sur deux cookies : `ZM_AUTH_TOKEN` pour un utilisateur et `ZM_ADMIN_AUTH_TOKEN` pour un administrateur, tous deux validés par mailboxd à chaque appel SOAP. Tout ce qui produit l'un de ces jetons sans le mot de passe du compte est un contournement complet. Trois voies reviennent : déléguer légitimement d'un jeton admin vers n'importe quel utilisateur (`DelegateAuthRequest`), forger un jeton utilisateur avec la clé **preauth** d'un domaine, et les failles de traitement des jetons où une requête forgée vers un point de terminaison privilégié renvoie directement un jeton utilisable. Le produit de cette page est un jeton ; remettez-le aux [chaînes de RCE](rce-chains.md) ou utilisez-le directement pour lire le courrier.

## Usurpation admin-vers-utilisateur (DelegateAuthRequest)

Un jeton admin est la clé maîtresse : le SOAP admin `DelegateAuthRequest` demande à mailboxd d'émettre un `ZM_AUTH_TOKEN` **utilisateur** pour tout compte nommé, par conception, afin qu'un administrateur puisse agir en tant qu'utilisateur. Si vous détenez un quelconque `ZM_ADMIN_AUTH_TOKEN` (admin obtenu par pulvérisation, ou forgé par une chaîne), vous pouvez vous connecter à chaque boîte aux lettres sans leurs mots de passe.

```http
POST /service/admin/soap HTTP/1.1
Host: target
Content-Type: application/soap+xml
Cookie: ZM_ADMIN_AUTH_TOKEN=<admin-token>

<soap:Envelope xmlns:soap="http://www.w3.org/2003/05/soap-envelope">
  <soap:Header>
    <context xmlns="urn:zimbra"><authToken><![CDATA[<admin-token>]]></authToken></context>
  </soap:Header>
  <soap:Body>
    <DelegateAuthRequest xmlns="urn:zimbraAdmin">
      <account by="name">victim@target</account>
    </DelegateAuthRequest>
  </soap:Body>
</soap:Envelope>
```

La réponse porte le jeton utilisateur délégué :

```xml
<DelegateAuthResponse>
  <authToken>0_abc...delegated...</authToken>
  <lifetime>3600000</lifetime>
</DelegateAuthResponse>
```

Définissez cette valeur comme `ZM_AUTH_TOKEN` contre `/service/soap` et vous êtes la victime : lisez, recherchez et envoyez en son nom. C'est l'étape canonique post-admin, et c'est pourquoi forger un jeton admin (ci-dessous, ou via le SSRF ProxyServlet dans [chaînes de RCE](rce-chains.md)) est l'objectif.

## Forge de jeton preauth

Zimbra prend en charge le **preauth** : un domaine détient une clé de preauth, et une URL vers `/service/preauth` portant un compte, un horodatage, une expiration et un HMAC de ces valeurs sur la clé connecte le compte sans mot de passe. Le mécanisme est destiné à l'authentification unique depuis un portail de confiance. Si la clé preauth du domaine fuit (depuis une lecture de configuration, une divulgation `GetDomainInfo`/`GetInfo`, ou une lecture de fichier antérieure), vous pouvez forger une connexion pour tout compte de ce domaine :

```bash
# preauth value = HMAC-SHA1( "account|by|expires|timestamp", preauth_key )
ts=$(date +%s000); exp=0
data="victim@target|name|$exp|$ts"
pa=$(printf '%s' "$data" | openssl dgst -sha1 -hmac "$PREAUTH_KEY" | awk '{print $2}')
curl -ski "https://target/service/preauth?account=victim@target&by=name&timestamp=$ts&expires=$exp&preauth=$pa"
```

Un `Set-Cookie: ZM_AUTH_TOKEN=...` dans la réponse est une session valide pour la victime, sans mot de passe. Toute la force du preauth est le secret de la clé, de sorte que toute divulgation de celle-ci est une primitive de prise de contrôle de compte à l'échelle du domaine.

## Failles de traitement des jetons (une requête forgée renvoie un jeton)

Plusieurs failles de Zimbra effacent la distinction entre une requête non authentifiée et une requête privilégiée, renvoyant ou acceptant un jeton qui n'aurait jamais dû être émis. Le schéma : une requête vers un point de terminaison privilégié est traitée comme préalablement de confiance parce qu'elle arrive via un chemin interne, une route proxifiée, ou un analyseur qui cadre mal le contexte d'authentification. La version la plus percutante s'atteint en routant une requête non authentifiée à travers le front end de sorte que mailboxd la voie comme un appel admin interne, implicitement de confiance, qui renvoie un jeton admin. Ce routage se fait avec le SSRF ProxyServlet et l'empoisonnement de route memcached documentés dans [chaînes de RCE](rce-chains.md) ; le résultat renvoyé ici est un `ZM_ADMIN_AUTH_TOKEN`, qui alimente ensuite le `DelegateAuthRequest` ci-dessus.

Lorsque le SOAP admin `AuthRequest` lui-même est joignable sans restriction réseau, un identifiant admin obtenu par pulvérisation produit directement le jeton admin :

```http
POST /service/admin/soap HTTP/1.1
Host: target
Content-Type: application/soap+xml

<soap:Envelope xmlns:soap="http://www.w3.org/2003/05/soap-envelope">
  <soap:Body>
    <AuthRequest xmlns="urn:zimbraAdmin">
      <name>admin@target</name><password>Sprayed1!</password>
    </AuthRequest>
  </soap:Body>
</soap:Envelope>
```

```xml
<AuthResponse><authToken>0_adminToken...</authToken><lifetime>43200000</lifetime></AuthResponse>
```

Un `<authToken>` dans une `AuthResponse` du namespace `urn:zimbraAdmin` est un jeton admin ; un `<soap:Fault>` signifie que l'identifiant ou la joignabilité a échoué.

## Notes d'exploitation

- La hiérarchie est **jeton admin -> tout jeton utilisateur** : `DelegateAuthRequest` est le pivot, de sorte que chaque chaîne vise d'abord un jeton admin.
- Un jeton utilisateur (`ZM_AUTH_TOKEN`) suffit pour une lecture complète de la boîte aux lettres et l'envoi au nom de ce compte ; un jeton admin (`ZM_ADMIN_AUTH_TOKEN`) suffit pour chaque compte et pour l'administration du serveur.
- La forge de preauth ne nécessite que la **clé preauth du domaine**, pas un secret par compte, de sorte qu'une fuite de clé est à l'échelle du domaine.
- Les jetons sont limités dans le temps par `lifetime` ; reforgez plutôt que de réutiliser un jeton périmé, et notez que les jetons admin sont de longue durée par défaut.

## Outils

- `openssl dgst -sha1 -hmac` pour calculer le HMAC de preauth.
- Burp Suite / curl pour rejouer les SOAP `AuthRequest` et `DelegateAuthRequest`.

## Références

- [Zimbra : mécanisme SSO preauth](https://wiki.zimbra.com/wiki/Preauth)
- [SOAP admin Zimbra (DelegateAuthRequest, AuthRequest)](https://wiki.zimbra.com/wiki/SOAP_API_Reference_Material_Beginning_with_ZCS_8.0)
- [Avis de sécurité Zimbra](https://wiki.zimbra.com/wiki/Zimbra_Security_Advisories)
