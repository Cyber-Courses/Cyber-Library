---
title: "macOS : attaquer la plateforme hôte macOS"
description: "Techniques offensives contre un hôte macOS : élévation de privilèges locale, contournement du framework de confidentialité TCC, de Gatekeeper et de la signature de code, et des frontières de System Integrity Protection et du bac à sable applicatif, lecture du trousseau, et persistance via les agents et démons de lancement."
keywords:
  - élévation de privilèges macOS
  - contournement de TCC
  - Gatekeeper
  - SIP
  - trousseau
---

# macOS

macOS superpose plusieurs de ses propres contrôles au-dessus d'une base Unix, de sorte qu'attaquer un Mac consiste autant à déjouer les frameworks de confidentialité, de signature et d'intégrité d'Apple qu'à pratiquer l'élévation de privilèges Unix classique. Le travail consiste à passer d'un utilisateur normal à `root`, à franchir les contrôles qui gouvernent les données sensibles et l'exécution de code, et à lire les secrets que le système protège.

## La surface locale

- **Élévation de privilèges** : les mécanismes Unix (SUID, `sudo`, chemins inscriptibles) ainsi que les abus spécifiques à macOS des services, des outils d'aide (helper tools) et des installateurs.
- **TCC** : contourner Transparency, Consent, and Control, le framework qui gouverne l'accès aux fichiers, à la caméra, au microphone et à l'automatisation, afin d'atteindre des données sans l'accord de l'utilisateur.
- **Gatekeeper et signature de code** : déjouer la quarantaine, la notarisation et les vérifications de signature pour exécuter du code non signé ou non fiable.
- **SIP et le bac à sable** : s'échapper du bac à sable applicatif et des frontières que System Integrity Protection impose même à `root`.
- **Accès aux identifiants** : lire les **trousseaux** de session et système et d'autres magasins d'identifiants.
- **Persistance** : agents et démons de lancement, éléments de connexion (login items), et profils de configuration.

## Références

- [Apple Platform Security guide](https://support.apple.com/guide/security/welcome/web)
- [MITRE ATT&CK: macOS](https://attack.mitre.org/matrices/enterprise/macos/)
