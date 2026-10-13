---
title: "Frontières de confiance applicatives : en-têtes de proxy, certificats client TLS et identité amont"
description: "Lorsque le code côté serveur fait confiance à des signaux d'identité fournis par le client ou en amont : en-têtes forwarded, mappage d'identité mTLS ou contexte de proxy, sans liaison saine à la requête."
keywords:
  - frontière de confiance
  - abus d'en-tête forwarded
  - reverse proxy
  - certificat client
---

# Frontière de confiance

Les défaillances de **frontière de confiance** se produisent lorsque le code **applicatif** prend des décisions d'autorisation ou de routage à partir de données qu'un attaquant peut influencer, parce qu'elles sont interprétées à tort comme provenant d'un chemin de confiance : en-têtes HTTP devant un saut inverse non sécurisé, identité client mTLS qui ne mappe pas de manière unique à un utilisateur, ou secret partagé entre services qui est devinable ou divulgué.

Le smuggling HTTP entrant (désaccord d'analyse entre sauts) est une famille différente sous **Injection → HTTP** ; cette page porte sur « qui l'application pense-t-elle que cette requête est ? » lorsque la réponse provient d'**en-têtes ou de certificats** auxquels l'application ne doit pas faire naïvement confiance depuis Internet.

## Pages

- [Abus de confiance des en-têtes de proxy](proxy-header-trust-abuse.md)
- [Mauvaise liaison de certificat client](client-certificate-misbinding.md)
