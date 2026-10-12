---
title: "Énumération : dresser l'empreinte d'un service IMAP et de ses utilisateurs"
description: "Énumérer IMAP avant authentification : la réponse CAPABILITY non étiquetée exposant LOGINDISABLED, STARTTLS et les mécanismes AUTH=, la bannière d'accueil identifiant Dovecot, Cyrus ou Courier, et la validation des noms d'utilisateur par différenciation des réponses LOGIN et AUTHENTICATE. Lit les réponses étiquetées OK/NO/BAD dans une session commentée."
keywords:
  - imap enumeration
  - imap capability
  - dovecot banner
  - imap user enumeration
  - loginDisabled
---

# Énumération

IMAP vous dit la plupart de ce dont vous avez besoin avant même de vous authentifier. La bannière d'accueil nomme fréquemment l'implémentation et le build, la réponse `CAPABILITY` indique si la connexion en clair est désactivée, si STARTTLS est requis et quels mécanismes SASL sont proposés, et la manière dont le serveur répond à une tentative de connexion pour un utilisateur connu par rapport à un inconnu peut révéler des noms d'utilisateur valides. Tout cela est non authentifié et chaque résultat sélectionne l'étape suivante : une liste de mécanismes à sprayer, une version à confronter aux failles d'implémentation, ou une liste d'utilisateurs à fournir à l'authentification.

## Dresser l'empreinte d'abord

Le serveur envoie un accueil non étiqueté (`* OK`) dès l'ouverture de la connexion, souvent avec la chaîne du produit. Puis un `CAPABILITY` étiqueté par le client liste les capacités du serveur. L'étiquette est n'importe quel jeton court que vous choisissez ; le serveur le reprend dans la dernière ligne.

```bash
nc <target> 143
* OK [CAPABILITY IMAP4rev1 SASL-IR LOGIN-REFERRALS ID ENABLE IDLE STARTTLS LOGINDISABLED AUTH=PLAIN] Dovecot ready.
A1 CAPABILITY
* CAPABILITY IMAP4rev1 SASL-IR LOGIN-REFERRALS ID ENABLE IDLE STARTTLS LOGINDISABLED AUTH=PLAIN
A1 OK Pre-login capabilities listed, post-login capabilities have more.
```

Lisez cela attentivement :

- `Dovecot ready.` identifie l'implémentation ; Cyrus répond par `* OK ... Cyrus IMAP ...`, Courier par `Courier-IMAP ready`.
- `LOGINDISABLED` signifie que la commande `LOGIN` (utilisateur/mot de passe en clair) est refusée sur ce canal en clair ; vous devez d'abord négocier `STARTTLS`, ou vous connecter au 993.
- `AUTH=PLAIN`, `AUTH=LOGIN`, `AUTH=CRAM-MD5`, `AUTH=NTLM` énumèrent les mécanismes SASL acceptés par `AUTHENTICATE`. `AUTH=NTLM` (ou `imap-ntlm-info`) divulgue les noms de domaine et d'hôte depuis le défi NTLM de type 2.

Sur 993, enveloppez le même échange dans TLS :

```bash
openssl s_client -connect <target>:993 -quiet
a CAPABILITY
```

## Validation des noms d'utilisateur

Là où le serveur renvoie une réponse sensiblement différente pour un nom d'utilisateur valide par rapport à un invalide, la surface de connexion devient un oracle d'utilisateurs. Deux signaux reviennent : la formulation du texte `a NO`, et le timing. Un serveur qui valide l'utilisateur avant de vérifier le mot de passe peut mettre un temps mesurablement plus long (ou formuler le rejet différemment) lorsque le compte existe.

```bash
openssl s_client -connect <target>:993 -quiet
a LOGIN alice wrongpass
a NO [AUTHENTICATIONFAILED] Authentication failed.
a LOGIN nosuchuser wrongpass
a NO [AUTHENTICATIONFAILED] Authentication failed.
```

Dovecot est délibérément uniforme ici : texte identique et délai artificiel dans les deux cas, de sorte que la différenciation est généralement morte. Les anciens Courier et les back ends SASL mal configurés (par exemple un frontal IMAP faisant office de proxy vers un back end qui répond « user unknown » plus vite que « bad password ») sont là où apparaissent les écarts de timing et de formulation. Scriptez l'écart de timing plutôt que de l'estimer à l'œil :

```bash
for u in $(cat users.txt); do
  t=$( { /usr/bin/time -f %e sh -c \
    "printf 'a LOGIN $u x\r\n' | openssl s_client -connect <target>:993 -quiet 2>/dev/null >/dev/null"; } 2>&1 )
  echo "$u $t"
done | sort -k2 -n        # outliers at either end are candidate valid users
```

Une sonde SASL `AUTHENTICATE` est l'autre oracle : certaines piles répondent à un utilisateur inconnu à l'étape du mécanisme plutôt qu'après l'identifiant, de sorte qu'un utilisateur inconnu échoue plus tôt dans l'échange qu'un utilisateur connu.

## Suite

Fournissez les noms d'utilisateur validés et la liste des mécanismes `AUTH=` à l'[authentification](authentication.md). Si la bannière a nommé un build Dovecot, Cyrus ou Courier spécifique, portez-le vers l'[exploitation serveur](server-exploitation.md).

## Outils

- [nmap imap-capabilities / imap-ntlm-info](https://nmap.org/nsedoc/scripts/imap-capabilities.html)

## Références

- [RFC 3501 : commande CAPABILITY](https://datatracker.ietf.org/doc/html/rfc3501#section-6.1.1)
- [RFC 3501 : LOGINDISABLED](https://datatracker.ietf.org/doc/html/rfc2595)
- [HackTricks : pentesting IMAP](https://book.hacktricks.wiki/en/network-services-pentesting/pentesting-imap.html)
