---
title: "Exposition sur Internet : accessibilité du bureau à distance via les relais de l'éditeur"
description: "Parce que les outils de bureau à distance se connectent vers un relais de l'éditeur, chaque client en cours d'exécution est atteignable depuis Internet sans port entrant, de sorte que la machine est exposée partout où l'attaquant peut atteindre le relais. Combiné à un ID énumérable et à un mot de passe faible ou fuité, cela fait de tout client installé une cible connectable à distance."
keywords:
  - exposition sur Internet
  - relais
  - accessibilité
  - identifiant d'appareil
  - connectable
---

# Exposition sur Internet

L'architecture de relais qui rend ces outils pratiques les rend aussi largement exposés. Un client se connecte vers l'extérieur au relais de l'éditeur et attend, de sorte qu'un attaquant qui connaît l'ID de l'appareil et le mot de passe se connecte par ce même relais depuis n'importe où, sans que la cible ait le moindre port entrant ouvert. La machine est donc de fait atteignable depuis Internet pour le contrôle à distance dès que le client s'exécute, indépendamment des pare-feux ou du NAT. Combiné aux [ID énumérables](../authentication/id-based-access.md) et aux [mots de passe faibles ou fuités](../authentication/weak-passwords.md), cela signifie que tout client installé est une cible connectable à distance, raison pour laquelle ces outils sont un vecteur courant d'accès initial et de persistance, y compris pour les opérations d'arnaque et de rançongiciel.

```bash
# the machine need not be internet-facing: the client reaches out to the relay,
# and the attacker connects via the relay using ID + password from anywhere.
# target sources: enumerated IDs, and ID:password pairs from breaches/stealer logs.
```

## Notes d'exploitation

- Le modèle de relais met en échec la segmentation réseau comme contrôle : un client sur une machine interne reste atteignable pour le contrôle à distance depuis Internet via le relais, de sorte que « non exposé sur Internet » n'est pas une protection pour ces outils.
- L'exigence pratique est l'ID plus le mot de passe ; les journaux d'infostealers fournissent couramment les deux pour une victime donnée, rendant l'exposition directement exploitable sans balayage.
- Cela sous-tend l'usage de ces outils dans les arnaques, la fraude au support et les rançongiciels : un client en cours d'exécution plus un mot de passe obtenu par ingénierie sociale ou fuité équivaut à un contrôle à distance.
- À associer aux [réglages par défaut non sécurisés](insecure-defaults.md) (accès sans surveillance, politique faible) qui rendent le client atteignable ouvert aux connexions.

## Références

- [TeamViewer / AnyDesk relay connectivity](https://www.teamviewer.com/en/)
- [CISA: remote access software abuse](https://www.cisa.gov/news-events/cybersecurity-advisories)
