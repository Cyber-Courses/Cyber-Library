---
order: 3
title: "Query : contournements d'allowlist SSRF au niveau paramètre"
description: "Lorsque l'URL récupérée est construite à partir de paramètres de requête, les redirections et la pollution de paramètres déjouent une allowlist après qu'elle a déjà approuvé la requête."
keywords:
  - paramètre de requête SSRF
  - open redirect SSRF
  - pollution de paramètres HTTP
  - contournement d'allowlist
  - paramètre url
---

# Query

De nombreux sinks de fetch tirent leur cible d'un paramètre de requête : `?url=`, `?target=`, `?image=`, `?next=`. La validation s'exécute généralement une fois, sur la chaîne de ce paramètre, puis la requête est émise. Les deux pages ici cassent l'hypothèse selon laquelle passer la validation une fois signifie que la requête finale est sûre.

- **[Contournement basé sur redirection](bypassing-using-a-redirect.md)** : l'URL soumise pointe vers un hôte que l'allowlist permet, mais cet hôte répond par une redirection vers une cible interne. Un client qui suit les redirections valide la première URL et se connecte à la seconde.
- **[Pollution de paramètres](parameter-pollution.md)** : fournir le paramètre deux fois (`?url=allowed&url=internal`) exploite un décalage entre le composant qui valide et le composant qui récupère, lorsqu'ils sont en désaccord sur l'occurrence qui l'emporte.

Les deux vivent à la couche query parce qu'elles manipulent la manière dont le paramètre est parsé et suivi, pas la manière dont l'hôte ou le chemin est écrit. Elles se combinent fréquemment avec les astuces d'[Authority](../authority/index.md) : la cible finale, post-redirection ou post-pollution, est toujours un hôte interne écrit avec l'un de ces encodages.

## Outils

- **Burp Suite** : manipuler les paramètres de style `?url=` et rejouer la requête construite.
- **SSRFmap** : automatiser la livraison de payloads de paramètre de requête contre le sink.
- **curl** : émettre des valeurs de paramètre dupliquées et redirigées pour observer la cible finale.

## Références

- [OWASP: Server Side Request Forgery](https://owasp.org/www-community/attacks/Server_Side_Request_Forgery)
- [PortSwigger: SSRF](https://portswigger.net/web-security/ssrf)
