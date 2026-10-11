---
title: "Énumération : comptes et version depuis le point de terminaison SOAP de Zimbra"
order: 1
description: "Énumérer Zimbra avant de détenir un jeton : valider les comptes via AuthRequest et les différences de connexion sur le point de terminaison public /service/soap, lire la version et la configuration via GetInfoRequest et GetVersionInfoRequest, sonder les chemins autodiscover et /home/, et détecter la console d'administration sur 7071. Enveloppes SOAP guidées avec interprétation des réponses."
keywords:
  - zimbra enumeration
  - zimbra soap authrequest
  - zimbra account enumeration
  - getversioninforequest
  - zimbra admin console detection
---

# Énumération

Zimbra répond à beaucoup de choses avant de vous authentifier. Le point de terminaison SOAP public à `/service/soap` accepte des enveloppes `AuthRequest` non authentifiées, et la manière dont il répond à un compte connu par rapport à un inconnu valide les noms d'utilisateur. La version et la configuration fuitent via les requêtes d'info SOAP et via les chemins d'assets client. La console d'administration sur 7071 est détectable indépendamment. Chaque résultat sélectionne l'étape suivante : une liste de comptes validés à pulvériser ou à alimenter dans `DelegateAuthRequest`, et une version exacte à faire correspondre aux [chaînes de RCE](rce-chains.md).

## Identifier la version

```bash
curl -sk https://<target>/zimbra/ | grep -oiE '/js/[0-9]+\.[0-9]+\.[0-9]+[^"]*' | head
curl -sk https://<target>:7071/zimbraAdmin/ -i | head        # admin console presence
```

Le chemin d'asset client intègre la version de ZCS ; un `/zimbraAdmin/` joignable sur 7071 confirme que la console d'administration est exposée au réseau, ce qui compte pour les chaînes de jeton admin. Un `GetVersionInfoRequest` public, non authentifié, renvoie directement la version là où il n'est pas restreint :

```http
POST /service/soap HTTP/1.1
Host: target
Content-Type: application/soap+xml

<soap:Envelope xmlns:soap="http://www.w3.org/2003/05/soap-envelope">
  <soap:Body>
    <GetVersionInfoRequest xmlns="urn:zimbraAccount"/>
  </soap:Body>
</soap:Envelope>
```

Un élément `<version>` renseigné dans le corps de la réponse épingle le niveau de correctif. Lisez-le contre la matrice des chaînes avant de lancer quoi que ce soit de spécifique à la version.

## Validation des comptes

`AuthRequest` est l'oracle de noms d'utilisateur. Envoyez une tentative de connexion pour un compte candidat avec un mot de passe bidon et lisez comment la faute SOAP diffère entre un compte qui existe et un qui n'existe pas. Les comptes connus renvoient une faute d'échec d'authentification (`account.AUTH_FAILED`) ; les comptes inconnus renvoient fréquemment une faute distincte ou un code/timing différent, et certaines configurations renvoient `account.NO_SUCH_ACCOUNT`.

```http
POST /service/soap HTTP/1.1
Host: target
Content-Type: application/soap+xml

<soap:Envelope xmlns:soap="http://www.w3.org/2003/05/soap-envelope">
  <soap:Body>
    <AuthRequest xmlns="urn:zimbraAccount">
      <account by="name">alice@target</account>
      <password>x</password>
    </AuthRequest>
  </soap:Body>
</soap:Envelope>
```

Interprétez le `<soap:Fault>` :

```xml
<soap:Fault>
  <soap:Code><soap:Value>soap:Sender</soap:Value></soap:Code>
  <soap:Reason><soap:Text>authentication failed for [alice@target]</soap:Text></soap:Reason>
  <soap:Detail><Error><Code>account.AUTH_FAILED</Code></Error></soap:Detail>
</soap:Fault>
```

Un `account.AUTH_FAILED` qui renvoie l'adresse signifie que le compte existe (le serveur a atteint la vérification du mot de passe). Une faute qui n'écho pas l'adresse, ou un code différent, signale un compte inexistant. Scriptez la liste de candidats et classez sur la valeur du `Code` et la présence de l'adresse renvoyée ; là où les codes sont uniformes, repliez-vous sur les écarts de timing de réponse comme avec tout oracle de connexion.

## Info post-jeton (et pourquoi l'énumération l'alimente)

Une fois un quelconque jeton utilisateur détenu (obtenu par pulvérisation, par hameçonnage, ou forgé via le [contournement d'authentification](authentication-bypass.md)), `GetInfoRequest` vide la configuration du compte, y compris le nom du serveur, les identifiants de source de données et l'ensemble de fonctionnalités, ce qui cartographie le back end pour l'étape de RCE :

```http
POST /service/soap HTTP/1.1
Host: target
Content-Type: application/soap+xml
Cookie: ZM_AUTH_TOKEN=<token>

<soap:Envelope xmlns:soap="http://www.w3.org/2003/05/soap-envelope">
  <soap:Header>
    <context xmlns="urn:zimbra"><authToken><![CDATA[<token>]]></authToken></context>
  </soap:Header>
  <soap:Body><GetInfoRequest xmlns="urn:zimbraAccount"/></soap:Body>
</soap:Envelope>
```

La réponse énumère la `<soapURL>` de l'hôte, le serveur de boîte aux lettres et les sources de données configurées, vous indiquant quel nœud de boîte aux lettres interne les chaînes ProxyServlet et `mboximport` doivent cibler.

## Autres surfaces

- **Autodiscover / `/Autodiscover/`** accepte une adresse e-mail et renvoie les paramètres de serveur par compte, validant les adresses et cartographiant les utilisateurs vers les hôtes back end.
- **`/home/<user>/`** et l'interface REST répondent différemment pour les boîtes aux lettres existantes par rapport aux manquantes, un oracle de comptes secondaire là où les fautes SOAP sont normalisées.
- **`/service/admin/soap` sur 443 et la console sur 7071** confirment que le plan d'administration est joignable, un prérequis pour les chaînes de jeton admin.

## Suites

Alimentez les comptes validés vers le [contournement d'authentification](authentication-bypass.md) (pulvérisation, ou comme cible `<account>` de `DelegateAuthRequest`) et portez la version exacte vers les [chaînes de RCE](rce-chains.md).

## Outils

- [nmap, curl pour les enveloppes SOAP brutes](https://curl.se/)
- Le repeater de Burp Suite pour itérer `AuthRequest` sur une liste de noms d'utilisateur.

## Références

- [Référence de l'API SOAP de Zimbra (AuthRequest, GetInfoRequest)](https://wiki.zimbra.com/wiki/SOAP_API_Reference_Material_Beginning_with_ZCS_8.0)
- [Zimbra : SOAP de provisionnement de comptes (namespace zimbraAdmin)](https://wiki.zimbra.com/wiki/Account_Provisioning)
- [HackTricks : pentesting des services web](https://book.hacktricks.wiki/en/network-services-pentesting/pentesting-web/index.html)
