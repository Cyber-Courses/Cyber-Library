---
title: "Authentification : pulvérisation SMTP AUTH et capture en clair"
description: "La soumission SMTP annonce les mécanismes AUTH (LOGIN, PLAIN, CRAM-MD5) dans sa réponse EHLO. PLAIN et LOGIN transportent des identifiants encodés en base64 trivialement interceptables sans TLS et pulvérisables en ligne, et un identifiant valide donne un relais authentifié et un accès à la boîte aux lettres en tant que véritable employé."
keywords:
  - smtp auth
  - AUTH PLAIN
  - AUTH LOGIN
  - password spraying
  - hydra smtp
---

# Authentification

Le service de soumission authentifie les clients avec la commande ESMTP `AUTH`, et les mécanismes qu'il accepte sont listés dans la réponse EHLO (`250-AUTH LOGIN PLAIN CRAM-MD5`). Deux de ces mécanismes comptent le plus pour un attaquant. `AUTH PLAIN` prend un unique jeton base64 de `\0username\0password`, de sorte que l'identifiant voyage en une ligne et se récupère par un décodage base64. `AUTH LOGIN` est la même idée en deux étapes : le nom d'utilisateur en base64, puis le mot de passe en base64. Aucun des deux n'est du chiffrement ; sur le port 25 ou 587 avant STARTTLS, l'identifiant est de fait en clair sur le réseau. Le serveur répond `235` en cas de succès et `535` en cas d'échec, un oracle net pour la devinette en ligne. Un identifiant SMTP fonctionnel est un véritable compte d'annuaire dans la plupart des déploiements, il est donc réutilisable bien au-delà de la messagerie.

## Prérequis

- EHLO annonce `AUTH`. S'il n'apparaît qu'après STARTTLS, exécutez d'abord `STARTTLS` (de nombreux serveurs masquent AUTH tant que le canal n'est pas chiffré).
- Préférez 587 (soumission) ou 465 (TLS implicite) ; le port 25 peut ne pas proposer AUTH du tout.
- Une liste d'utilisateurs validée issue de l'[énumération des utilisateurs](user-enumeration.md) pour que les pulvérisations atteignent de vrais comptes.

## Construire et envoyer AUTH à la main

```bash
# AUTH PLAIN token is base64 of NUL user NUL pass
printf '\0jsmith\0Spring2026!' | base64
#   AGpzbWl0aABTcHJpbmcyMDI2IQ==
```
```text
EHLO attacker.test
250-AUTH LOGIN PLAIN
AUTH PLAIN AGpzbWl0aABTcHJpbmcyMDI2IQ==
235 2.7.0 Authentication successful        # 235 = valid credential
# wrong password:
535 5.7.8 Error: authentication failed     # 535 = rejected
```
`235` est un identifiant actif ; `535` est un échec. Lire une ligne `AUTH PLAIN` capturée sur le réseau est l'opération inverse : `echo AGpzbWl0aABTcHJpbmcyMDI2IQ== | base64 -d | tr '\0' ':'` affiche `:jsmith:Spring2026!`.

## Pulvérisation

```bash
# one password across the validated user list, submission port
hydra -L users.txt -p 'Spring2026!' -s 587 mail.victim.com smtp
# swaks to test a single pair and watch the full dialogue
swaks --server mail.victim.com:587 --auth LOGIN --auth-user jsmith --auth-password 'Spring2026!' --tls
```
Hydra n'affiche `[587][smtp] host: ... login: jsmith password: Spring2026!` que sur un `235`. Gardez un faible nombre de mots de passe par tour : les services de soumission journalisent les échecs d'authentification et verrouillent les comptes, et dans une organisation adossée à Entra une rafale déclenche le verrouillage intelligent.

## Suites

- Relais authentifié : une fois connecté, le serveur envoie au nom du domaine de votre compte avec une autorisation complète, de sorte que le courrier passe SPF/DKIM/DMARC et arrive comme un véritable expéditeur interne. C'est la voie propre vers le hameçonnage par fraude au président sans avoir besoin d'un [open relay](open-relay.md).
- Le même identifiant débloque généralement la boîte aux lettres via IMAP/POP/OWA et l'annuaire au sens large ; pulvérisez-le immédiatement contre d'autres services.
- Pour Microsoft 365, l'AUTH SMTP hérité est souvent désactivé ; pulvérisez plutôt la plateforme d'identité, voir [pulvérisation de mots de passe Entra ID](../../../../../online/identity/entra-id/authentication/password-spraying.md).

## Outils

- [hydra](https://github.com/vanhauser-thc/thc-hydra)
- [swaks](https://github.com/jetmore/swaks)

## Références

- [RFC 4954 (SMTP Service Extension for Authentication)](https://datatracker.ietf.org/doc/html/rfc4954)
- [RFC 4616 (le mécanisme SASL PLAIN)](https://datatracker.ietf.org/doc/html/rfc4616)
- [HackTricks : SMTP (25,465,587)](https://book.hacktricks.wiki/en/network-services-pentesting/pentesting-smtp/index.html)
