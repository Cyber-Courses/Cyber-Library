---
title: "Downgrade d'échange de clés : groupes d'échange de clés VPN faibles"
description: "Les VPN négociant des paramètres d'échange de clés faibles, petits groupes Diffie-Hellman (768/1024 bits) ou groupes IKE hérités, laissent un attaquant bien positionné ou disposant de ressources importantes attaquer l'accord de clés. Forcer ou observer un groupe faible permet des attaques par précalcul contre les paramètres partagés et le déchiffrement des clés de session qui en dérivent."
keywords:
  - key exchange
  - diffie-hellman
  - dh group
  - logjam
  - downgrade
---

# Downgrade d'échange de clés

Un VPN dérive ses clés de session via un échange de clés, et des paramètres faibles rendent cet accord attaquable. IPsec/IKE négocie un groupe Diffie-Hellman parmi les transforms proposés, et les gateways qui offrent encore de petits groupes (DH group 1 = 768 bits, group 2 = 1024 bits) sont exposées : ces groupes sont à portée d'attaques par précalcul contre leurs paramètres partagés fixes (la classe Logjam), après quoi les échanges de clés individuels qui les utilisent sont cassés à un coût relativement faible. Un downgrade, forcé ou la gateway acceptant simplement un petit groupe, fait tomber la session à ce niveau attaquable, et les clés de session dérivées (et donc le tunnel) tombent avec lui.

```bash
# which DH groups does the IKE gateway offer?
ike-scan -M <target>                            # transform output includes the DH group
#   group 1 (768) / group 2 (1024) => small, precomputation-vulnerable
# force a small group to confirm acceptance, or observe a handshake that used one;
# attacking the fixed 1024-bit group parameters (Logjam-style) then breaks the exchange
```

## Notes d'exploitation

- Les groupes DH 1 et 2 offerts (768/1024 bits) sont le constat ; leurs paramètres fixes sont susceptibles d'un précalcul unique qui casse ensuite à bas coût tout échange qui les utilise, de sorte qu'une gateway qui les accepte est attaquable par un adversaire disposant de ressources importantes.
- Le downgrade s'applique là où la gateway accepte un groupe faible aux côtés de groupes forts ; un attaquant force la sélection faible ou attend un client qui la négocie.
- Cela attaque l'accord de clés lui-même (livrant les clés de session), distinct des [chiffrements faibles](weak-ciphers.md) qui attaquent le chiffrement de masse ; les deux aboutissent à un trafic de tunnel déchiffrable.
- Associez à une position de capture ; le gain pratique est le déchiffrement d'un tunnel capturé qui a utilisé le groupe faible.

## Références

- [Logjam / weak Diffie-Hellman](https://weakdh.org/)
- [NIST SP 800-77: IKE groups](https://csrc.nist.gov/pubs/sp/800/77/r1/final)
