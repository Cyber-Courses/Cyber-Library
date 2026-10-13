---
order: 3
title: "SSRF via proxy_pass variable nginx : sélection d'amont influencée par l'utilisateur"
description: "Exploitation d'un proxy_pass nginx construit à partir de données de requête : des amonts variables qui désactivent la validation et laissent un attaquant rediriger la requête mandatée vers des services internes et les métadonnées cloud."
keywords:
  - ssrf proxy_pass
  - proxy_pass variable
  - ssrf nginx
  - métadonnées cloud
  - gopher
---

# SSRF via proxy_pass variable

Quand nginx construit son amont `proxy_pass` à partir de données de requête, un attaquant redirige la requête mandatée vers un service interne, transformant l'edge de confiance en véhicule de SSRF. Comme la requête provient de nginx (à l'intérieur du réseau), elle atteint des hôtes et des ports que l'attaquant ne peut pas toucher directement.

## Le motif dangereux

Utiliser une variable dans `proxy_pass` désactive la résolution-au-démarrage normale de l'amont par nginx et peut incorporer l'entrée de l'attaquant :

```nginx
location /proxy/ {
    proxy_pass http://$host$request_uri;      # Host chooses the upstream HOST (port dropped)
}
location /raw/ {
    proxy_pass http://$http_host$request_uri; # $http_host keeps the port from the Host header
}
location /fetch/ {
    set $target $arg_url;
    proxy_pass http://$target;                # ?url=169.254.169.254/... -> full host:port control
}
```

Un détail qui piège les gens : `$host` est le nom de serveur normalisé **sans** le port, donc `Host: 127.0.0.1:6379` avec `$host` se connecte à `127.0.0.1:80`, pas à Redis sur 6379. Pour contrôler le port depuis l'en-tête Host, il faut `$http_host` (la valeur brute de l'en-tête), et pour contrôler librement l'hôte et le port, la forme `$arg_url` est la plus simple :

```
GET /fetch/?url=127.0.0.1:6379/ HTTP/1.1
GET /fetch/?url=169.254.169.254/latest/meta-data/iam/security-credentials/ HTTP/1.1
GET /raw/ HTTP/1.1
Host: 127.0.0.1:9000
```

Cibles classiques : métadonnées cloud (`169.254.169.254`), panneaux d'administration réservés à localhost et API HTTP internes (Elasticsearch, Consul, Docker, Kubernetes, actuator).

## Problèmes de chemin et de jointure

Même sans amont variable, un décalage de slash final entre `location` et `proxy_pass`, ou des caractères transmis non échappés, peuvent altérer le chemin amont, atteignant des routes backend non prévues. Des CRLF ou espaces encodés que nginx laisse passer ont, sur certaines versions, été injectés dans la ligne de requête amont.

## La portée est HTTP(S) uniquement

Le `proxy_pass` HTTP de nginx ne parle que HTTP et HTTPS, cette primitive ne peut donc pas émettre d'octets bruts vers un service non-HTTP : vous ne pouvez pas transformer un `proxy_pass` variable en un payload FastCGI/Redis/SMTP `gopher://`. Il atteint les services **HTTP** internes et les métadonnées cloud, ce qui est déjà à fort impact. Les pivots en octets bruts (un paquet FastCGI vers [PHP-FPM](fastcgi-and-php-fpm.md), un flux de commandes Redis) requièrent un sink de SSRF *différent* dont le client d'URL supporte `gopher://`, comme un `curl`/fetch de bibliothèque au niveau applicatif, pas nginx lui-même. De nombreux services internes (Redis, Memcached) restent joignables ici s'ils tolèrent une ligne de requête HTTP, mais la route propre en protocole brut nécessite le sink compatible gopher.

## Exploitation

- Trouvez les endpoints de proxy qui prennent une URL/un hôte/un chemin (`/proxy`, `/fetch`, `/render`, fetchers d'image ou de webhook) et pointez-les vers `127.0.0.1`, les métadonnées link-local et les noms d'hôte internes.
- Lisez les métadonnées cloud pour les identifiants, puis pivotez hors de l'hôte.
- Confirmez une SSRF aveugle avec une interaction hors bande (Burp Collaborator), puis ciblez des services internes spécifiques.

## Outils

- Burp Repeater/Collaborator (confirmer la SSRF aveugle hors bande). Gopherus ne s'applique qu'à un sink de SSRF compatible gopher distinct, pas au `proxy_pass` de nginx.

## Références

- nginx : proxy_pass (réserves sur la forme variable)
- PortSwigger Web Security Academy : SSRF
