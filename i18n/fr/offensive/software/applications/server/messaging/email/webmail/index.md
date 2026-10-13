---
title: "Webmail : attaquer le front web au-dessus d'un back end IMAP et SMTP"
order: 3
description: "Le webmail est une application HTTP posée sur un magasin de courrier IMAP/SMTP, il est donc attaqué comme une application web (XSS stocké dans le rendu des messages, SSRF, contournement d'authentification, écriture de fichier vers RCE) et comme un back end de courrier joignable derrière lui. Identifiez le produit depuis la page de connexion, puis naviguez vers Roundcube, Horde ou Zimbra."
keywords:
  - webmail
  - roundcube
  - horde
  - zimbra
  - stored xss webmail
---

# Webmail

Un produit de webmail est une application web qui rend le courrier depuis un magasin IMAP et l'envoie via un relais SMTP. Tout ce qui le rend utile est aussi la surface d'attaque : il analyse et affiche du courrier HTML fourni par l'attaquant, il récupère du contenu distant, il stocke les préférences et plugins par utilisateur sur le disque, et il détient des cookies de session de longue durée vers des boîtes aux lettres. Un hôte de webmail est donc attaqué sur deux plans à la fois. Le **plan web** est l'application PHP ou Java elle-même : XSS stocké dans le rendu des messages qui se déclenche quand la victime ouvre votre courrier, falsification de requête côté serveur depuis les récupérations d'images distantes ou de calendrier, failles d'authentification et de traitement de jetons, et écritures de fichiers sous la racine web qui deviennent une exécution de code. Le **plan courrier** est le back end IMAP/SMTP vers lequel le front relaie, joignable une fois que vous disposez d'une session ou d'un identifiant de back end.

La propriété décisive est que vous délivrez la charge utile par e-mail. Contrairement à une application web normale où il vous faut que la victime visite un lien, un XSS stocké dans le rendu des messages s'exécute au moment où la cible lit un message que vous lui avez envoyé, avec sa session, à l'intérieur de l'origine de sa boîte aux lettres. Cela transforme un seul message forgé en vol de session, envoi silencieux au nom de la victime, et exfiltration de la boîte aux lettres.

## Identifier quel webmail

Identifiez le produit et la version avant toute chose : le chemin d'exploitation est entièrement spécifique au produit, et la version décide quel contournement du rendu des messages ou quelle chaîne d'écriture de fichier s'applique.

```bash
# Login-page and path fingerprints
curl -sk https://<target>/ -i | grep -iE 'server:|x-powered-by|set-cookie'
curl -sk https://<target>/?_task=login -i        # Roundcube: redirects/renders its login
curl -sk https://<target>/horde/ -i              # Horde: /horde/ application root
curl -sk https://<target>/zimbra/ -i             # Zimbra web client
curl -sk https://<target>:7071/zimbraAdmin/ -i   # Zimbra admin console (separate port)
```

Lisez les signaux :

- **Roundcube** : l'URL de connexion porte `?_task=login`, le cookie de session est `roundcube_sessid` (et `roundcube_sessauth`), et le balisage référence `program/js/` et `skins/`. La version est dans `CHANGELOG.md` à la racine web sur beaucoup d'installations, ou dans la requête d'asset `?s=<hash>`.
- **Horde** : une racine d'application à `/horde/`, le cookie `Horde`, le formulaire de connexion postant vers `/horde/login.php`, et le module de webmail nommé **IMP**. Une `<meta name="generator">` ou l'arborescence `/horde/services/` le confirme.
- **Zimbra** : le client sous `/zimbra/`, la console d'administration sur **7071** à `/zimbraAdmin/`, les points de terminaison SOAP `/service/soap` et `/service/admin/soap`, les cookies `ZM_AUTH_TOKEN` et `ZM_ADMIN_AUTH_TOKEN`. Les chaînes de version apparaissent dans les chemins d'assets `/zimbra/js/`.
- **Favicon et generator** : hachez le favicon (`curl -sk https://<target>/favicon.ico | md5sum`) contre une table de produits connus, et cherchez au grep dans le HTML `<meta name="generator">`, que Roundcube et Horde émettent tous deux.

Une fois le produit et la version connus, allez à la page correspondante et partez de sa propre section d'identification.

## Pages

- **[Roundcube](roundcube.md)** : XSS stocké dans le rendu des messages HTML (contournements de `rcube_washtml`), traversée de chemin dans les skins et plugins, et les chaînes d'écriture de fichier vers PHP.
- **[Horde](horde.md)** : CSRF contre les points de terminaison admin et préférences, injection de préférences et de formulaires, et l'injection de commandes via le gestionnaire d'images vers RCE.

## Sous-rubriques

- **[Zimbra](zimbra/index.md)** : la Collaboration Suite complète, de l'énumération de comptes et l'abus de jeton admin à l'écriture de web shell `mboximport` et la chaîne SSRF ProxyServlet vers admin.

## Références

- [HackTricks : pentesting web (interfaces webmail)](https://book.hacktricks.wiki/en/network-services-pentesting/pentesting-web/index.html)
- [PortSwigger Web Security Academy : cross-site scripting](https://portswigger.net/web-security/cross-site-scripting)
- [Blog de recherche SonarSource (analyses de vulnérabilités de webmail)](https://www.sonarsource.com/blog/)
