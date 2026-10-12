---
title: "Mobile : attaquer les applications Android et iOS"
description: "Techniques offensives contre les applications mobiles : stockage local non sécurisé de secrets, composants exportés et communication interprocessus non sécurisée, abus de liens profonds et de schémas d'URL, transport faible et épinglage de certificat, et contournement du bac à sable de la plateforme ainsi que des vérifications de root/jailbreak et d'altération sur Android et iOS."
keywords:
  - mobile application security
  - Android
  - iOS
  - insecure storage
  - deep links
---

# Mobile

Une application mobile s'exécute dans un bac à sable (sandbox) propre à chaque application, sur un appareil que l'utilisateur transporte, et une grande partie de sa surface d'attaque réside dans ce qu'elle stocke localement, ce qu'elle expose aux autres applications de l'appareil, et la manière dont elle communique avec son backend. La position de l'attaquant va d'une autre application sur le même appareil à un utilisateur qui contrôle (et peut rooter ou jailbreaker) son propre téléphone pour inspecter et altérer l'application.

## La surface

- **Stockage non sécurisé** : secrets, jetons et données personnelles laissés dans les préférences partagées, les bases de données, les fichiers ou le keychain/keystore sans protection adéquate.
- **Communication interprocessus** : composants Android exportés (activités, services, broadcast receivers, content providers) et schémas d'URL iOS et extensions d'application que d'autres applications peuvent atteindre ou détourner.
- **Liens profonds et schémas d'URL** : liens qui amènent l'application dans des états privilégiés ou font passer des données clandestinement lorsqu'ils sont ouverts depuis l'extérieur.
- **Transport** : configuration TLS faible et contournement de l'épinglage de certificat qui exposent ou altèrent le trafic.
- **Intégrité de la plateforme** : défaite de la détection de root/jailbreak, des vérifications de débogueur et d'altération, et du bac à sable qui confine l'application.

## Jonctions

L'API côté serveur de l'application mobile est attaquée en tant que cible [Serveur](../../server/index.md) ; ce domaine concerne l'application sur l'appareil.

## Références

- [OWASP Mobile Application Security (MASVS/MASTG)](https://mas.owasp.org/)
- [OWASP Mobile Top 10](https://owasp.org/www-project-mobile-top-10/)
