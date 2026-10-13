---
order: 2
title: "Cryptographie faible : faiblesses de chiffrement et d'échange de clés du VPN"
description: "Les gateways VPN qui négocient des chiffrements ou des paramètres d'échange de clés faibles, chiffrements hérités, petits groupes Diffie-Hellman, SHA-1, laissent un attaquant bien positionné sélectionner ou rétrograder vers une cryptographie cassable. Cela permet le déchiffrement du trafic de tunnel capturé ou des attaques sur l'accord de clés, sapant la confidentialité pour laquelle le VPN existe."
keywords:
  - vpn crypto
  - cipher
  - key exchange
  - diffie-hellman
  - downgrade
---

# Cryptographie faible

La raison d'être d'un VPN est la confidentialité, donc sa cryptographie négociée est la cible. Les gateways qui offrent encore des options faibles, chiffrements hérités (DES/3DES, RC4), petits groupes Diffie-Hellman (768/1024 bits) ou intégrité SHA-1, donnent à un attaquant bien positionné une session cassable : la négociation peut aboutir, ou être rétrogradée, vers l'option faible, après quoi le trafic de tunnel capturé est déchiffrable ou l'accord de clés est attaquable. Cela compte surtout pour IPsec/IKE (où les transform sets sont explicitement proposés) et pour les VPN basés sur TLS avec des suites de chiffrement faibles, et cela aggrave les faiblesses d'authentification et de protocole.

```bash
# IKE transform sets offered (weak DH groups, DES/3DES, SHA-1?)
ike-scan -M <target>
# TLS-based VPN (SSL-VPN/SSTP/OpenVPN-TLS) cipher grading
nmap -p443 --script ssl-enum-ciphers <target>
```

## Sous-thèmes

- **[Chiffrements faibles](weak-ciphers.md)** : négociation de chiffrements hérités et cassés.
- **[Downgrade d'échange de clés](key-exchange-downgrade.md)** : groupes d'échange de clés petits ou faibles.

## Références

- [NIST SP 800-77: VPN crypto](https://csrc.nist.gov/pubs/sp/800/77/r1/final)
- [Logjam / weak DH](https://weakdh.org/)
