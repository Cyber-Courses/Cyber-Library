---
order: 1
title: "Authority : ciblage de l'hôte et du port dans le SSRF"
description: "L'authority est l'hôte et le port sur lesquels une requête atterrit. Les cibles loopback et link-local, les encodages d'IP alternatifs, les astuces de résolution de nom d'hôte et la portée de port vivent tous ici."
keywords:
  - authority SSRF
  - localhost
  - 169.254.169.254
  - encodage d'IP
  - scan de ports
  - contournement d'allowlist d'hôtes
---

# Authority

Le composant authority d'une URL est le `host:port` qui décide *où* la requête atterrit. C'est la partie qu'un validateur veut le plus contraindre, car autoriser un hôte arbitraire est ce qui transforme un fetch en SSRF, et c'est aussi la partie avec le plus de contournements, car un hôte peut être écrit de nombreuses façons équivalentes et résolu à un moment du choix de l'attaquant.

Cibler l'authority recouvre trois manœuvres :

- **Atteindre des hôtes internes.** La loopback (`127.0.0.1`, `[::1]`), toutes les interfaces (`0.0.0.0`), les plages RFC1918 et l'adresse de métadonnées link-local cloud `169.254.169.254` sont les destinations à forte valeur. Les pages ci-dessous les atteignent même lorsque les chaînes littérales sont bloquées.
- **Écrire l'hôte pour qu'un filtre le manque.** Les encodages d'[adresse IP](ip-address.md) (décimal, octal, hexadécimal, IPv6-mappé, raccourci) et les astuces au niveau nom d'hôte sous [Nom de domaine](domain-name/index.md) (DNS rebinding, domaines internationalisés) résolvent tous vers une cible interne tout en se lisant comme autre chose.
- **Choisir le port.** [Port](port.md) couvre le balayage et la sélection de ports, ce qui transforme un fetch aveugle en scanner et oriente des payloads de scheme ultérieurs vers le bon service.

## Le point faible du validateur

Les allowlists et blocklists d'hôtes opèrent sur une chaîne, mais le réseau se connecte à une IP résolue. Chaque page de ce sous-arbre élargit la faille entre les deux : la chaîne que le validateur inspecte et l'adresse que le socket atteint réellement. Un contrôle qui compare l'hôte littéral, normalise différemment du client HTTP, ou résout le nom une fois pour la validation et de nouveau pour la connexion est déjoué par l'une des techniques ici.

## Outils

- **SSRFmap** : exploitation SSRF automatisée contre les cibles internes courantes et les contournements d'authority.
- **Burp Collaborator** : détecter la portée d'un SSRF aveugle via des callbacks DNS et HTTP out-of-band.
- **curl** : sondage manuel des hôtes, ports et schemes pour confirmer quelles formes d'authority se connectent.

## Références

- [OWASP: Server Side Request Forgery](https://owasp.org/www-community/attacks/Server_Side_Request_Forgery)
- [PortSwigger: SSRF](https://portswigger.net/web-security/ssrf)
