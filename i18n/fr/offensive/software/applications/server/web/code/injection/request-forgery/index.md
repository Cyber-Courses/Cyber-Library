---
order: 6
title: "Request forgery : abus d'URL côté serveur (SSRF)"
description: "Lorsqu'une entrée non fiable oriente une requête sortante, un attaquant atteint des services internes, des métadonnées cloud et des schemes d'URL alternatifs. Ce hub décompose l'attaque par facette d'URL et par le client qui effectue le fetch."
keywords:
  - SSRF
  - server-side request forgery
  - sink d'URL
  - métadonnées cloud
  - schemes d'URL alternatifs
  - accès à un service interne
---

# Request forgery

La request forgery survient lorsqu'une application prend une URL, un hôte ou un chemin dans une entrée non fiable et émet une requête sortante vers elle. Le serveur devient un proxy qui parle depuis l'intérieur du périmètre réseau, de sorte qu'une requête que l'attaquant ne pourrait jamais envoyer directement atteint désormais les services loopback, les endpoints de métadonnées link-local, les plans de contrôle cloud et les voisins sur le réseau interne. La server-side request forgery (SSRF) est le nom courant, mais la même primitive pilote les lectures de fichiers via `file://`, l'interaction TCP brute via `gopher://` et la contrebande de protocole via les handlers qu'une bibliothèque d'URL se trouve enregistrer.

La fonctionnalité qui l'expose est presque toujours bénigne : un émetteur de webhooks, un aperçu ou déroulement de lien, un proxy d'avatar ou d'image, un moteur de rendu PDF ou de capture d'écran, un champ import-depuis-URL, un health check ou un convertisseur de documents. Chacun d'eux prend une URL et la récupère, et chacun est un sink.

## Comment cette section est organisée

Une URL a des parties distinctes, et un validateur fait généralement confiance à certaines tandis qu'une attaque est possible à travers d'autres. Les sous-arbres suivent ces parties :

- **[Authority](authority/index.md)** : l'hôte et le port. Cibles loopback et link-local, encodages d'IP alternatifs, astuces de nom d'hôte (DNS rebinding, domaines internationalisés) et portée de port.
- **[Path](path.md)** : traversée et normalisation à l'intérieur du chemin d'URL, utilisées pour échapper à une URL de base fixe ou pour confondre une allowlist qui ne vérifie qu'un préfixe.
- **[Query](query/index.md)** : contournements au niveau paramètre, y compris les redirections de second ordre et la pollution de paramètres HTTP qui déjouent une allowlist après qu'elle a déjà laissé passer.
- **[Scheme](scheme/index.md)** : le handler de protocole. Au-delà de `http(s)`, une bibliothèque d'URL peut honorer `file`, `gopher`, `dict`, `ldap`, `ftp`/`sftp`, `tftp`, `jar` et `netdoc`, chacun atteignant une classe de cible différente.

Orthogonal à la grammaire de l'URL se trouve **[Fetch](fetch/index.md)**, la question de *comment* la pile effectue la requête : un client HTTP programmatique, ou un navigateur headless complet qui transporte des cookies et exécute du script. La même URL malveillante se comporte différemment dans chacun, donc le client est son propre axe.

## Pourquoi cela escalade

Un simple fetch qui ne renvoie rien de visible reste utile : le timing de réponse et les différences d'erreur le transforment en scanner de ports et en oracle de services. Lorsque la réponse *est* reflétée, le service de métadonnées renvoie des identifiants d'instance, les panneaux d'administration internes renvoient leur contenu, et `file://` renvoie des fichiers locaux. Lorsque le scheme autorise des octets bruts sur le fil, `gopher://` écrit une commande Redis ou SMTP confectionnée et convertit une primitive de lecture en exécution de code ou en relais de courrier. Les facettes ci-dessous sont les routes vers chacune de ces issues.

## Outils

- **SSRFmap** : exploitation SSRF automatisée contre les cibles loopback, de métadonnées et internes.
- **Burp Suite** : confectionner et rejouer les requêtes de fetch en échangeant les facettes d'URL et les clients.
- **Burp Collaborator** : confirmer les SSRF aveugles et out-of-band via des callbacks DNS et HTTP.
- **interactsh** : serveur d'interaction OOB auto-hébergé pour détecter les requêtes sortantes aveugles.

## Références

- [OWASP: Server Side Request Forgery](https://owasp.org/www-community/attacks/Server_Side_Request_Forgery)
- [OWASP WSTG: Testing for Server-Side Request Forgery](https://owasp.org/www-project-web-security-testing-guide/latest/4-Web_Application_Security_Testing/07-Input_Validation_Testing/19-Testing_for_Server-Side_Request_Forgery)
- [PortSwigger: SSRF](https://portswigger.net/web-security/ssrf)
- [PayloadsAllTheThings: Server Side Request Forgery](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Server%20Side%20Request%20Forgery)
