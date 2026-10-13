---
order: 2
title: "Usurpation d'IP : se faire passer pour un hôte de confiance auprès de rsh"
description: "Les r-commands s'authentifient par IP source contre les fichiers de confiance, donc usurper l'adresse d'un hôte de confiance peut satisfaire la vérification sans le contrôler. Parce que rsh/rlogin ont besoin des réponses du serveur, l'attaque requiert soit une position on-path, soit la prédiction des numéros de séquence TCP, l'attaque de spoofing aveugle classique contre la confiance basée sur l'adresse."
keywords:
  - ip spoofing
  - trusted host
  - tcp sequence prediction
  - blind spoofing
  - rsh
---

# Usurpation d'IP

Les r-commands décident de la confiance à partir de l'IP source du client (résolue contre `.rhosts`/`hosts.equiv`), donc un attaquant qui fait en sorte que ses paquets semblent provenir d'un hôte de confiance satisfait la vérification sans posséder cet hôte. La complication est que rsh/rlogin sont TCP et interactifs, donc l'attaquant a normalement besoin des réponses du serveur. Deux voies résolvent cela : une position on-path (voir les réponses directement), ou le spoofing aveugle, prédire les numéros de séquence initiaux TCP du serveur pour achever le handshake et injecter la commande sans voir les réponses, l'attaque classique que la confiance basée sur l'adresse invite.

```bash
# on-path: spoof the trusted source and relay/observe responses, then:
rsh -l root <target> 'echo "+ +" >> /root/.rhosts'    # as the trusted host, open a backdoor
# blind spoofing (historic): predict the target's TCP ISN to forge a full connection
#   from the trusted host's address and inject a command (e.g. writing .rhosts),
#   converting a one-shot spoof into durable passwordless access
```

## Notes d'exploitation

- L'objectif n'est généralement pas une session en direct (difficile en aveugle) mais l'injection d'une seule commande qui établit une confiance durable, classiquement `echo "+ +" >> ~/.rhosts` pour l'utilisateur cible, après quoi l'attaquant se connecte normalement.
- Le spoofing on-path est simple là où vous pouvez voir le trafic de l'hôte de confiance ; le spoofing aveugle dépend de numéros de séquence TCP prévisibles, que les piles modernes randomisent, donc il est surtout viable contre les vieux systèmes (exactement là où vivent les r-commands).
- C'est l'attaque que la confiance basée sur l'adresse rend fondamentalement possible ; elle ne nécessite aucun identifiant ni accès en écriture, seulement la capacité de forger une source de confiance et de faire exécuter une commande.
- Combinez avec la connaissance des hôtes de confiance (depuis un [hosts.equiv](hosts-equiv.md) ou `.rhosts` lisible) pour choisir une adresse de confiance usurpable.

## Références

- [Morris/Bellovin TCP sequence prediction](https://www.rfc-editor.org/rfc/rfc1948)
- [HackTricks: rsh trust](https://book.hacktricks.xyz/network-services-pentesting/pentesting-rsh)
