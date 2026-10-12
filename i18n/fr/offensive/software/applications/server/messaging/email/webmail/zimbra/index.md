---
title: "Zimbra : attaquer le client web, la console d'administration et le SOAP de la Collaboration Suite"
order: 3
description: "La Zimbra Collaboration Suite comme surface d'attaque : l'application mailboxd (Jetty) derrière un proxy nginx, le client web sur 443 et la console d'administration sur 7071, et l'API SOAP à /service/soap et /service/admin/soap. Identifiez la version, puis progressez à travers l'énumération de comptes, l'abus de jeton admin et les chaînes de RCE non authentifiées."
keywords:
  - zimbra
  - zimbra collaboration suite
  - zimbra soap
  - mailboxd
  - zimbra admin console
---

# Zimbra

La Zimbra Collaboration Suite (ZCS) est une plateforme de messagerie et de groupware dont le côté serveur est une application Java, **mailboxd**, s'exécutant dans Jetty, précédée d'un proxy **nginx** et appuyée par **memcached** (cache de routes et d'authentification), **amavis/clamav** (analyse du courrier) et MariaDB. Les utilisateurs atteignent le client web sur 443 sous `/zimbra/` ; les administrateurs utilisent une console séparée sur **7071** sous `/zimbraAdmin/`. Presque chaque fonction serveur, utilisateur comme admin, est pilotée par une API **SOAP** : opérations utilisateur à `/service/soap`, opérations privilégiées à `/service/admin/soap`, avec l'authentification portée par les cookies `ZM_AUTH_TOKEN` (utilisateur) et `ZM_ADMIN_AUTH_TOKEN` (admin).

C'est cette architecture qui fait de Zimbra une cible riche. Le proxying nginx-vers-mailboxd, le modèle de jeton SOAP et les nombreuses extensions (`mboximport`, Autodiscover, le ProxyServlet) ont chacun produit des chemins joignables, souvent non authentifiés, vers un web shell ou un jeton admin. L'API SOAP fait aussi fuiter des informations de compte avant l'authentification, offrant une étape d'énumération nette qui alimente tout ce qui suit.

## Triage

Confirmez Zimbra et épinglez la version ; les chaînes de RCE sont strictement liées à la version.

```bash
curl -sk https://<target>/zimbra/ -i | grep -iE 'set-cookie|location'   # user client
curl -sk https://<target>:7071/zimbraAdmin/ -i | head                   # admin console
curl -sk https://<target>/service/admin/soap -i | head                  # admin SOAP endpoint
# version from client asset paths
curl -sk https://<target>/zimbra/ | grep -oE 'skins|/js/[0-9]+\.[0-9]+[^"]*' | head
```

Les cookies `ZM_AUTH_TOKEN`/`ZM_ADMIN_AUTH_TOKEN`, les points de terminaison `/service/soap` et `/service/admin/soap`, et la console d'administration sur 7071 confirment ensemble ZCS. La chaîne de version apparaît dans les chemins d'assets JS du client et, avec un accès local, via `zmcontrol -v`. Notez le niveau de correctif exact : il décide laquelle des chaînes de RCE ci-dessous est active.

## Pages

- **[Énumération](enumeration.md)** : découverte de comptes et de version via le point de terminaison SOAP public et les différences de connexion avant de détenir un jeton.
- **[Contournement d'authentification](authentication-bypass.md)** : obtenir ou forger un jeton d'authentification valide, y compris l'usurpation admin-vers-utilisateur via `DelegateAuthRequest` et les failles de traitement des jetons qui forgent un jeton sans identifiants.
- **[Chaînes de RCE](rce-chains.md)** : l'écriture de web shell `mboximport` non authentifiée, l'injection de réponse CRLF memcached, le SSRF ProxyServlet vers le SOAP admin, et l'XXE Autodiscover, chacune aboutissant à l'exécution de code sur le serveur.

## Références

- [HackTricks : pentesting Zimbra / services web](https://book.hacktricks.wiki/en/network-services-pentesting/pentesting-web/index.html)
- [Référence de l'API SOAP de Zimbra](https://wiki.zimbra.com/wiki/SOAP_API_Reference_Material_Beginning_with_ZCS_8.0)
- [Avis de sécurité Zimbra](https://wiki.zimbra.com/wiki/Zimbra_Security_Advisories)
