---
title: "Énumération : identifier un service POP3 et ses utilisateurs"
description: "Énumérer POP3 avant l'authentification : la bannière du message d'accueil identifiant qpopper, Courier ou Dovecot et exposant l'horodatage APOP, la réponse CAPA listant STLS/SASL/UIDL, et la validation des noms d'utilisateur par différences de réponses USER/PASS et de timing. Lit +OK et -ERR dans une session guidée."
keywords:
  - pop3 enumeration
  - capa
  - apop timestamp
  - pop3 user enumeration
  - stls
---

# Énumération

POP3 révèle son identité dès la première ligne et ses capacités en une seule commande. La bannière du message d'accueil nomme souvent l'implémentation, et si le serveur prend en charge APOP, il y intègre un jeton d'horodatage par connexion directement dans le message d'accueil. `CAPA` indique si STARTTLS (`STLS`) est proposé, quels mécanismes SASL existent, et si `UIDL`/`TOP` sont disponibles. Le duo `USER`/`PASS` est aussi un oracle de noms d'utilisateur sur les serveurs qui distinguent « utilisateur inexistant » de « mauvais mot de passe ». Tout cela est non authentifié.

## Identifier d'abord la version

Le serveur envoie un message d'accueil `+OK` à la connexion. Sur un serveur compatible APOP, le message d'accueil contient un jeton `<...>`, l'horodatage utilisé pour le condensé APOP.

```bash
nc <target> 110
+OK POP3 mail.corp.local v2020.1 server ready <1896.697170952@mail.corp.local>
CAPA
+OK Capability list follows
TOP
USER
UIDL
SASL PLAIN LOGIN
STLS
APOP
.
```

Lisez ceci :

- La chaîne du message d'accueil nomme fréquemment le produit (`qpopper`, `Courier`, `Dovecot ready`). Un `<token@host>` présent dans le message d'accueil signifie qu'APOP est proposé et vous donne le défi si vous souhaitez l'attaquer.
- `STLS` signifie que STARTTLS est disponible sur le port 110 ; son absence sur un port en clair signifie que `USER`/`PASS` traverseront le réseau en clair (utile à savoir pour l'interception, et pour déterminer si le serveur impose TLS).
- `SASL` liste les mécanismes `AUTH` ; `UIDL` et `TOP` confirment que vous pouvez lister les identifiants uniques et jeter un œil aux en-têtes lors de l'[accès à la boîte aux lettres](mailbox-access.md).

Sur le port 995, enveloppez-le dans TLS :

```bash
openssl s_client -connect <target>:995 -quiet
CAPA
```

## Validation des noms d'utilisateur

Les implémentations POP3 classiques répondent à `USER` seul par `+OK` quoi qu'il arrive, puis révèlent l'existence du compte à l'étape `PASS` ou via le timing. Lorsque le serveur (ou le back end SASL derrière lui) renvoie une erreur distincte pour un utilisateur inconnu, la connexion devient un oracle d'utilisateurs.

```bash
nc <target> 110
USER bob
+OK
PASS x
-ERR [AUTH] Authentication failed.          # account exists, bad password
USER nobody
+OK
PASS x
-ERR [SYS/PERM] No such user.                # account does not exist (older servers)
```

Dovecot moderne normalise les deux cas vers le même message et ajoute un délai, supprimant la différence de formulation ; repliez-vous donc sur le timing : un serveur qui valide l'utilisateur avant de hacher le mot de passe répond plus lentement à un compte connu qu'à un compte inconnu.

```bash
for u in $(cat users.txt); do
  t=$( { /usr/bin/time -f %e sh -c \
    "printf 'USER $u\r\nPASS x\r\nQUIT\r\n' | openssl s_client -connect <target>:995 -quiet 2>/dev/null >/dev/null"; } 2>&1 )
  echo "$u $t"
done | sort -k2 -n
```

qpopper hérité et les anciens Courier sont les endroits où la formulation explicite « utilisateur inexistant » survit ; triez la sortie de timing pour les serveurs modernes à message uniforme.

## Suites

Transmettez les noms d'utilisateur validés ainsi que le jeton APOP et la liste `SASL` à l'[authentification](authentication.md), et le produit/la version de la bannière à l'[exploitation du serveur](server-exploitation.md).

## Outils

- [nmap pop3-capabilities / pop3-ntlm-info](https://nmap.org/nsedoc/scripts/pop3-capabilities.html)

## Références

- [RFC 1939 : APOP et l'horodatage du message d'accueil](https://datatracker.ietf.org/doc/html/rfc1939#section-7)
- [RFC 2449 : commande CAPA de POP3](https://datatracker.ietf.org/doc/html/rfc2449)
- [HackTricks : pentesting POP](https://book.hacktricks.wiki/en/network-services-pentesting/pentesting-pop.html)
