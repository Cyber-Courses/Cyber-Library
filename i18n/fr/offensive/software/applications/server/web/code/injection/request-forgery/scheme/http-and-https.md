---
title: "HTTP et HTTPS : le schéma SSRF de base"
description: "Les schémas http et https par défaut atteignent les services web internes, les endpoints de santé et de debug, et l'API de métadonnées du cloud, et constituent le point de départ avant d'escalader vers d'autres schémas."
keywords:
  - http SSRF
  - https SSRF
  - métadonnées cloud
  - service web interne
  - 169.254.169.254
---

# HTTP et HTTPS

`http` et `https` sont les schémas pour lesquels un sink de récupération est conçu, et l'essentiel de la valeur d'une SSRF est accessible sans jamais changer de schéma. L'attaquant se contente de rediriger la requête vers une cible web interne au lieu de la cible externe prévue.

## Cibles web internes

Tout service web que le serveur peut atteindre est dans le périmètre, y compris ceux liés à la loopback ou à un réseau interne et jamais exposés publiquement :

```
http://127.0.0.1:8080/admin
http://localhost:8161/            # internal management consoles
http://10.0.0.5/actuator/env      # framework debug and management endpoints
http://internal-api.corp/v1/keys
```

Les endpoints de gestion et de debug ont une grande valeur car ils sont souvent non authentifiés lorsqu'ils sont atteints depuis localhost, et ils exposent la configuration, les variables d'environnement et les identifiants.

## Le service de métadonnées du cloud

Sur une instance cloud, l'endpoint de métadonnées link-local renvoie l'identité de l'instance et, fréquemment, des identifiants temporaires :

```
http://169.254.169.254/latest/meta-data/
http://169.254.169.254/latest/meta-data/iam/security-credentials/
http://169.254.169.254/latest/user-data/
```

Lorsque la réponse est reflétée, le document d'identifiants revient directement. Des équivalents existent chez les différents fournisseurs à la même adresse `169.254.169.254` et à des noms d'hôte spécifiques au fournisseur, nécessitant parfois un en-tête particulier, ce qui compte lorsque le sink permet à l'attaquant d'influencer les en-têtes.

## Rôle dans l'arbre des schémas

HTTP est le schéma de base et la première chose à essayer, mais il s'agit d'une lecture de ce que le service renvoie. Lorsqu'un service découvert nécessite une requête multiligne élaborée ou une commande brute (Redis, SMTP, un `POST` avec un corps précis), escaladez vers [Gopher](gopher.md). Pour les fichiers locaux, passez à [File](file.md). Associez le sondage HTTP au balayage de [Port](../authority/port.md) pour trouver les services qui méritent qu'on y pointe un payload de schéma, et aux encodages d'[Authority](../authority/index.md) lorsque l'adresse interne est filtrée.

## Outils

- **SSRFmap** : exploitation automatisée de SSRF sur les cibles web internes et l'endpoint de métadonnées du cloud.
- **Burp Collaborator** : détection d'interactions hors-bande pour les SSRF aveugles.
- **curl** : sondage manuel des cibles internes `http://` et `https://` et des URL de métadonnées.

## Références

- [OWASP: Server Side Request Forgery](https://owasp.org/www-community/attacks/Server_Side_Request_Forgery)
- [PortSwigger: SSRF](https://portswigger.net/web-security/ssrf)
