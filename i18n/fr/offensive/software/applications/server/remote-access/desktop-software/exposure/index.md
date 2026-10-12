---
title: "Exposition : outils de bureau à distance exposés sur Internet et mal configurés"
description: "Les outils de bureau à distance atteignent les machines via les relais de l'éditeur, de sorte qu'ils fonctionnent depuis n'importe où, ce qui, combiné à une configuration faible ou par défaut, expose ces machines largement. La surface se compose des installations accessibles depuis Internet, détectables par énumération, et des réglages par défaut non sécurisés, accès sans surveillance activé, politique de mot de passe faible, absence de liste d'autorisation, qui laissent un client ouvert aux connexions."
keywords:
  - exposition
  - exposé sur Internet
  - réglages par défaut non sécurisés
  - accès sans surveillance
  - relais
---

# Exposition

De par leur conception, ces outils se connectent via le relais cloud de l'éditeur, de sorte que tout client installé est atteignable depuis n'importe où l'attaquant peut atteindre ce relais, ce qui revient de fait à Internet, sans que la machine elle-même soit directement exposée sur Internet. Cette portée, combinée à une configuration faible, constitue l'exposition. Les deux aspects sont les installations accessibles depuis Internet (tout client en cours d'exécution dont l'ID est connu et le mot de passe faible ou fuité est connectable depuis l'extérieur) et les réglages par défaut non sécurisés, accès sans surveillance activé, politique de mot de passe faible ou inexistante, absence de liste d'autorisation de connexion ou d'exigence d'approbation, qui laissent le client ouvert. Le modèle de relais signifie que « interne uniquement » n'est pas une véritable frontière pour ces outils.

## Sous-thèmes

- **[Exposition sur Internet](internet-exposure.md)** : accessibilité via le relais de l'éditeur.
- **[Réglages par défaut non sécurisés](insecure-defaults.md)** : configurations qui laissent un client ouvert.

## Références

- [TeamViewer / AnyDesk connectivity model](https://www.teamviewer.com/en/)
- [HackTricks](https://book.hacktricks.xyz/)
