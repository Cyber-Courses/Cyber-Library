---
order: 1
title: "Usurpation IP : se faire passer pour une source de confiance des r-commands"
description: "Parce que la confiance r-command est vérifiée contre l'IP source, usurper l'adresse d'un hôte de confiance satisfait la vérification sans le contrôler. Avec une position sur le chemin, l'attaquant voit les réponses directement ; en aveugle, il prédit les numéros de séquence TCP pour compléter la connexion et injecter une commande, l'attaque classique contre la confiance basée sur l'adresse."
keywords:
  - usurpation ip
  - adresse source
  - prédiction de séquence tcp
  - hôte de confiance
  - usurpation en aveugle
---

# Usurpation IP

La confiance basée sur l'hôte est évaluée à partir de l'IP source de la connexion : forger cette adresse pour correspondre à un hôte de confiance vainc donc la vérification sans posséder l'hôte. Sur l'ensemble des r-commands, c'est l'attaque qu'invite l'authentification basée sur l'adresse. Avec une position sur le chemin, l'attaquant usurpe la source de confiance et observe les réponses du serveur, complétant un échange interactif. En aveugle, sans voir les réponses, l'attaquant prédit les numéros de séquence initiaux TCP du serveur pour forger une connexion complète depuis l'adresse de confiance et injecter une seule commande, classiquement une qui établit une confiance durable pour que l'accès ultérieur ne nécessite plus d'usurpation.

```bash
# identify a trusted host to impersonate (from a readable hosts.equiv/.rhosts)
cat /etc/hosts.equiv ~*/.rhosts 2>/dev/null
# on-path: spoof the trusted source, relay/observe responses, then run a command
rsh -l root <target> 'echo "+ +" >> /root/.rhosts'    # as the trusted host
# blind: predict the target TCP ISN, forge the connection from the trusted address,
#   and inject one command (e.g. writing .rhosts) to convert the spoof into durable access
```

## Notes d'exploitation

- L'objectif habituel est d'injecter une seule commande qui implante une confiance durable ([écriture rhosts](rhosts-write.md)), après quoi l'attaquant se connecte normalement sans usurpation.
- L'usurpation sur le chemin est facile lorsque vous voyez le trafic de l'hôte de confiance ; l'usurpation en aveugle nécessite des numéros de séquence TCP prévisibles, que les piles modernes randomisent, elle n'est donc viable que contre les anciens systèmes sur lesquels tournent les r-commands.
- Ne nécessite ni identifiant ni accès en écriture, seulement une source de confiance forgée et une commande exécutée ; c'est l'expression la plus pure de la faiblesse de confiance basée sur l'adresse.
- L'adresse de confiance précise à forger provient de la lecture de `hosts.equiv`/`.rhosts` ou de la déduction des relations de confiance.

## Références

- [RFC 1948 (defending against sequence prediction)](https://www.rfc-editor.org/rfc/rfc1948)
- [HackTricks: r-commands](https://book.hacktricks.xyz/network-services-pentesting/pentesting-rsh)
