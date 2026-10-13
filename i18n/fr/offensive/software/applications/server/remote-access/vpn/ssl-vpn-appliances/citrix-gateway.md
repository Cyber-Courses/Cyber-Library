---
order: 4
title: "Citrix Gateway : chaîne traversée-vers-RCE et la divulgation de jeton Citrix Bleed"
description: "Citrix Gateway et ADC (NetScaler) ont connu une chaîne de path traversal donnant l'exécution de code à distance non authentifiée, et la faille de divulgation mémoire Citrix Bleed qui fait fuiter des jetons de session depuis l'appliance, permettant à un attaquant de détourner des sessions authentifiées et de contourner l'authentification multifacteur. Les deux ont été exploitées à grande échelle pour l'accès initial."
keywords:
  - citrix
  - netscaler
  - adc
  - citrix bleed
  - session token
---

# Citrix Gateway

Citrix Gateway et ADC (NetScaler) servent de façade à l'accès distant et aux applications, et deux classes de failles en ont fait des cibles de choix. Une vulnérabilité de path traversal chaînée vers l'exécution de code à distance non authentifiée sur l'appliance, donnant le contrôle de l'équipement. Et Citrix Bleed est une faille de divulgation mémoire : une requête non authentifiée vers la gateway renvoie de la mémoire non initialisée qui contient des jetons de session valides, de sorte qu'un attaquant lit l'endpoint de façon répétée, récolte des jetons de session et les rejoue pour détourner des sessions utilisateur authentifiées, contournant à la fois les mots de passe et l'authentification multifacteur. Les deux ont été exploitées à grande échelle, Citrix Bleed notamment par des groupes de ransomware, pour l'accès initial.

```bash
# fingerprint Citrix Gateway / NetScaler ADC
curl -skI https://<gateway>/
curl -sk https://<gateway>/vpn/index.html | grep -i netscaler
# Citrix Bleed class: repeatedly request the vulnerable endpoint; the response leaks
# memory containing session tokens -> replay a token to hijack an authenticated session
for i in $(seq 1 200); do curl -sk 'https://<gateway>/<bleed-endpoint>' ; done | strings | grep -i 'sessionid\|token'
# traversal-to-RCE class: a crafted path reaches a writable/exec location -> RCE
```

## Notes d'exploitation

- Citrix Bleed est caractéristique : elle ne nécessite aucun identifiant et déjoue la MFA, parce que les jetons de session fuités correspondent à des sessions déjà authentifiées ; rejouer un jeton récolté place l'attaquant dans la session authentifiée de cet utilisateur sur la gateway.
- La récolte est une attaque par répétition : chaque requête fait fuiter un fragment de mémoire, donc de nombreuses requêtes collectent des jetons ; filtrez les réponses à la recherche de motifs de jeton de session.
- La classe traversée-vers-RCE donne directement l'exécution de code sur l'appliance ; identifiez à quelle faille la version cible est exposée.
- Les deux donnent un accès qui sert de façade aux applications internes et au réseau ; Citrix Bleed en particulier a alimenté de vastes intrusions par ransomware via des sessions détournées.

## Références

- [Citrix security bulletins](https://support.citrix.com/securitybulletins)
- [CISA: Citrix Bleed exploitation](https://www.cisa.gov/news-events/cybersecurity-advisories)
