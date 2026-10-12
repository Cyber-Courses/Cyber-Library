---
title: "Réglages par défaut non sécurisés : configurations de bureau à distance qui laissent un client ouvert"
description: "Les clients de bureau à distance sont fréquemment laissés dans des configurations non sécurisées : accès sans surveillance activé avec un mot de passe faible, absence de liste d'autorisation de connexion ou d'invite d'approbation, politique de mot de passe faible et authentification supplémentaire désactivée. Ces réglages, combinés à la portée Internet de l'outil, laissent la machine ouverte aux connexions de quiconque obtient l'ID et le mot de passe."
keywords:
  - réglages par défaut non sécurisés
  - accès sans surveillance
  - liste d'autorisation
  - invite d'approbation
  - configuration
---

# Réglages par défaut non sécurisés

La configuration de ces clients détermine à quel point une machine atteignable est ouverte, et les réglages non sécurisés sont courants. Un accès sans surveillance activé avec un mot de passe faible, réutilisé ou jamais changé offre une connectivité permanente. L'absence d'une liste d'autorisation de connexion (restreignant quels ID peuvent se connecter) ou d'une invite d'approbation signifie que tout client disposant de l'ID et du mot de passe se connecte sans obstacle. Une politique de mot de passe faible autorise des mots de passe courts ou devinables, et les protections supplémentaires (double facteur, approbation d'appareil) ne sont souvent pas activées. Combinés à la [portée Internet](internet-exposure.md) de l'outil fondée sur le relais, ces réglages par défaut laissent la machine ouverte aux connexions de quiconque obtient l'ID et le mot de passe.

```bash
# on a host, review the client configuration for the open settings
#   unattended access enabled? allowlist configured? approval required? 2FA on?
#   (locations: TeamViewer/AnyDesk registry and app-data config)
reg query 'HKLM\SOFTWARE\WOW6432Node\TeamViewer' 2>nul      # inspect settings
# weak unattended password + no allowlist + no 2FA = open to anyone with ID+password
```

## Notes d'exploitation

- Le réglage par défaut le plus risqué est l'accès sans surveillance avec un mot de passe faible ou réutilisé et sans liste d'autorisation : c'est un contrôle à distance permanent et sans obstacle pour quiconque détient l'identifiant, voir l'[accès sans surveillance](../authentication/unattended-access.md).
- L'absence de liste d'autorisation de connexion et d'invite d'approbation signifie qu'il n'y a pas de second verrou au-delà du mot de passe ; là où le double facteur ou l'approbation d'appareil est disponible mais désactivé, le mot de passe est la seule barrière.
- Inspectez ces réglages sur tout hôte que vous atteignez ; un client configuré de façon permissive est à la fois une voie d'accès et un mécanisme de persistance.
- Ces réglages par défaut sont ce qui rend exploitable l'[exposition sur Internet](internet-exposure.md) de l'outil ; ensemble, ils expliquent le fort détournement de ces produits pour l'accès à distance.

## Références

- [TeamViewer hardening/allowlist](https://www.teamviewer.com/en/trust-center/security/)
- [AnyDesk security settings](https://anydesk.com/en/security)
