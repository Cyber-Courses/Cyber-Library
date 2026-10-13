---
order: 5
title: "L2TP et SSTP : faiblesses héritées de la PSK IPsec et de TLS"
description: "L2TP ne fournit lui-même aucun chiffrement et est associé à IPsec, de sorte qu'il hérite des faiblesses de clé pré-partagée IPsec, une PSK L2TP/IPsec faible ou par défaut est capturée et cassée comme n'importe quelle PSK IKE. SSTP tunnelise PPP sur TLS, de sorte qu'il hérite des faiblesses de configuration TLS : chiffrements faibles, downgrade, et lacunes de validation de certificat permettant l'interception."
keywords:
  - l2tp
  - sstp
  - psk ipsec
  - tls
  - ppp
---

# L2TP et SSTP

L2TP et SSTP sont tous deux portés par autre chose, et ils héritent des faiblesses de ce porteur. L2TP ne fournit aucune confidentialité par lui-même et est presque toujours déployé en L2TP/IPsec, où IPsec fournit le chiffrement à l'aide d'une clé pré-partagée ; cette PSK est fréquemment faible, par défaut ou largement partagée, et est capturée et cassée exactement comme n'importe quelle PSK IKE, après quoi les informations d'identification utilisateur (PPP/MS-CHAP à l'intérieur) sont la cible suivante. SSTP tunnelise PPP sur une connexion TLS (TCP 443), de sorte que sa sécurité est celle de TLS : des suites de chiffrement faibles, un downgrade de protocole, et surtout des certificats non vérifiés ou auto-signés permettent à un attaquant bien positionné d'intercepter, et l'authentification PPP interne (souvent MS-CHAPv2) est alors exposée.

```bash
# L2TP/IPsec: attack the IPsec PSK first (aggressive mode, if offered)
ike-scan -A -M -P psk.hash <target>; psk-crack -d wordlist.txt psk.hash
# SSTP (TLS on 443): grade the TLS and check certificate validation
nmap -p443 --script ssl-enum-ciphers <target>
openssl s_client -connect <target>:443 | openssl x509 -noout -issuer -subject
# inner PPP/MS-CHAPv2 (both protocols) is crackable if captured (see PPTP)
```

## Notes d'exploitation

- La sécurité de L2TP est entièrement celle d'IPsec : une PSK L2TP/IPsec faible/par défaut est la même attaque qu'[IPsec IKE](ipsec-ike.md) et [clés pré-partagées faibles](../authentication/weak-pre-shared-keys.md) ; une PSK unique partagée par tous les utilisateurs est courante et constitue le point d'ancrage.
- La sécurité de SSTP est entièrement celle de TLS : des certificats non vérifiés/auto-signés et des chiffrements faibles permettent le MITM et l'interception, évaluez-le donc comme tout point de terminaison [TLS faible](../weak-cryptography/weak-ciphers.md).
- Les deux portent couramment MS-CHAPv2 à l'intérieur de la couche PPP, qui est cassable à partir d'une négociation capturée comme [PPTP](pptp.md) ; de sorte que même après la couche externe, l'authentification interne est attaquable.
- Leur nature héritée fait que ces protocoles sont rarement originaux : attaquez le porteur (PSK IPsec ou TLS), puis les informations d'identification PPP internes.

## Références

- [RFC 3931 (L2TPv3) et L2TP/IPsec](https://datatracker.ietf.org/doc/html/rfc3931)
- [Présentation de SSTP par Microsoft](https://learn.microsoft.com/en-us/windows-server/remote/remote-access/)
