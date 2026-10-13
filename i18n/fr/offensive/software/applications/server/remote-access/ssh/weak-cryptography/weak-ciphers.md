---
order: 3
title: "Chiffrements faibles : faiblesses des chiffrements SSH hérités et en mode CBC"
description: "Les serveurs SSH qui proposent encore des chiffrements hérités, single-DES, RC4/arcfour, ou des chiffrements en mode CBC, exposent la session à des faiblesses connues. Le mode CBC dans SSH a été sujet à une attaque de récupération de texte clair, et les chiffrements de flux et hérités sont cryptographiquement faibles, de sorte qu'un serveur qui les annonce permet à un attaquant positionné d'affaiblir ou d'altérer le canal chiffré."
keywords:
  - chiffrement faible
  - mode cbc
  - rc4 arcfour
  - 3des
  - chiffrement ssh
---

# Chiffrements faibles

SSH chiffre la session avec un chiffrement négocié, et les vieux serveurs en proposent encore de faibles. Les chiffrements en mode CBC (par exemple `aes128-cbc`, `3des-cbc`) ont été sujets à une attaque classique de récupération de texte clair dans SSH qui exploitait la manière dont CBC interagit avec la gestion de la longueur des paquets par SSH, permettant à un attaquant de récupérer un texte clair limité à partir d'une session observée. Le chiffrement de flux `arcfour` (RC4) est cryptographiquement cassé, et le single-DES est trivialement faible. Un serveur annonçant l'un de ces éléments signale une configuration faible et, pour un attaquant capable de capturer ou de se positionner sur la connexion, un chemin vers l'affaiblissement ou l'altération du canal.

```bash
# which ciphers are offered?
nmap -p22 --script ssh2-enum-algos <target> | grep -A20 encryption_algorithms
# force a weak/CBC cipher to confirm acceptance
ssh -c aes128-cbc user@<target>
ssh -c 3des-cbc user@<target>
ssh -c arcfour user@<target>        # RC4, if offered
```

## Notes d'exploitation

- La liste `encryption_algorithms` proposée est l'indice : toute entrée `-cbc`, `arcfour*`, `3des` ou `des` marque une configuration de serveur faible.
- La faiblesse de récupération de texte clair en CBC nécessite une position on-path ou de capture et des conditions spécifiques ; son rendement pratique est un texte clair limité, mais elle signale un serveur qui mérite d'être attaqué cryptographiquement plutôt que seulement par l'authentification.
- Les chiffrements faibles sont rarement isolés ; un serveur qui les propose propose aussi généralement des MAC et un KEX faibles, aggravant l'exposition, voir [algorithmes MAC faibles](weak-mac-algorithms.md) et [downgrade d'échange de clés](key-exchange-downgrade.md).
- Même là où l'exploitation est impraticable, la présence de chiffrements faibles identifie un build SSH ancien, probablement vulnérable, qu'il vaut la peine de vérifier pour des CVE d'implémentation.

## Références

- [SSH CBC plaintext recovery (CPNI advisory)](https://www.openssh.com/txt/cbc.adv)
- [RFC 4253: encryption](https://datatracker.ietf.org/doc/html/rfc4253)
