---
title: "Chiffrements faibles : négociation de chiffrements VPN hérités et cassés"
description: "Les gateways VPN offrant des chiffrements hérités, DES simple ou triple, RC4, ou des suites de niveau export, laissent un attaquant bien positionné négocier ou rétrograder vers un chiffrement assez faible pour déchiffrer le trafic de tunnel capturé. IPsec expose cela via ses transform sets proposés et les VPN basés sur TLS via leurs suites de chiffrement."
keywords:
  - weak cipher
  - 3des
  - rc4
  - transform set
  - decryption
---

# Chiffrements faibles

Un VPN qui offre encore un chiffrement faible sape sa propre confidentialité. Dans IPsec, la gateway annonce des transform sets, et l'acceptation de DES, 3DES ou de modes faibles laisse une session utiliser un chiffrement cassable. Dans les VPN basés sur TLS (portails SSL-VPN, SSTP, OpenVPN-sur-TLS), les suites de chiffrement faibles (RC4, niveau export, chiffrements par blocs de 64 bits vulnérables aux attaques par anniversaire) font de même. Un attaquant bien positionné qui capture le tunnel, ou qui force la négociation vers l'option faible, peut alors déchiffrer ou affaiblir le trafic que le VPN est censé protéger.

```bash
# IPsec: which transforms are offered (look for DES/3DES, weak hashes)
ike-scan -M <target>
# TLS VPN: grade the cipher suites
nmap -p443 --script ssl-enum-ciphers <target>        # flags RC4, 3DES, export, SWEET32-prone
# capture tunnel traffic for offline analysis where a weak suite is in use
```

## Notes d'exploitation

- Les transform sets proposés (IPsec) et les suites de chiffrement (VPN TLS) sont la preuve ; l'acceptation de DES/3DES, RC4 et des suites de niveau export marque une gateway dont le trafic est attaquable moyennant une capture.
- Les chiffrements par blocs de 64 bits (DES/3DES) sont vulnérables aux attaques par collision à seuil d'anniversaire sur des tunnels longue durée (la classe SWEET32), pertinentes parce que les tunnels VPN transportent de gros volumes sur de longues sessions.
- La cryptographie faible compte pour un attaquant capable de capturer ou de se placer sur le tunnel ; combinez avec une position de capture et avec le [downgrade d'échange de clés](key-exchange-downgrade.md) qui attaque l'accord de clés lui-même.
- Même quand le déchiffrement est impraticable, la prise en charge de chiffrements faibles identifie une gateway ancienne probablement vulnérable aux failles d'[appliance](../ssl-vpn-appliances/index.md) ou de protocole.

## Références

- [SWEET32 (64-bit block cipher attacks)](https://sweet32.info/)
- [nmap ssl-enum-ciphers](https://nmap.org/nsedoc/scripts/ssl-enum-ciphers.html)
