---
title: "Authentification de l'expéditeur : usurper au-delà de SPF, DKIM et DMARC"
order: 1
description: "Comment les destinataires vérifient SPF, DKIM et DMARC sur le courrier entrant, et comment un attaquant lit les enregistrements publiés pour trouver la faille qui permet à un message d'usurper un domaine cible dans l'en-tête From: visible tout en passant. Oriente vers les pages enfants SPF, DKIM, DMARC et usurpation d'expéditeur."
keywords:
  - sender authentication
  - SPF
  - DKIM
  - DMARC
  - email spoofing
  - DMARC alignment
---

# Authentification de l'expéditeur

Un serveur de messagerie destinataire décide si un message entrant provient vraiment du domaine qu'il prétend en exécutant trois contrôles contre les enregistrements DNS que le domaine expéditeur publie. SPF demande si l'adresse IP d'envoi est autorisée pour le domaine d'enveloppe `MAIL FROM` (et `HELO`). DKIM vérifie une signature cryptographique sur des en-têtes sélectionnés et le corps contre une clé publique que le domaine publie. DMARC relie les deux à l'unique identité qu'un humain voit réellement, l'en-tête `From:`, en exigeant qu'une identité SPF ou DKIM passante *s'aligne* avec le domaine du `From:`, et il déclare quoi faire quand aucune ne s'aligne.

L'objectif de l'attaquant est de délivrer un message dont le `From:` visible est un domaine cible (afin que le destinataire lui fasse confiance) et qui franchit tout de même ces contrôles, ou qui n'est pas soumis à une politique bloquante. Chaque voie vers ce résultat est une faille dans l'un des trois enregistrements, de sorte que le premier geste contre toute cible est de récupérer les trois et de les lire. L'asymétrie clé à garder en tête : SPF et DKIM authentifient des identités que le destinataire ne voit jamais (l'expéditeur d'enveloppe, le domaine signataire), et seul DMARC lie quelque chose à l'en-tête `From:`. Un domaine avec un SPF et un DKIM parfaits mais sans DMARC appliqué est grand ouvert à l'usurpation du `From:`.

## Triage

Récupérez les trois enregistrements et lisez la politique avant toute chose. Le sélecteur DKIM n'est pas devinable en général, prenez-le dans le tag `DKIM-Signature: ... s=` de tout courrier que vous avez reçu de la cible, ou forcez des noms courants (voir [DKIM](dkim.md)).

```bash
dig +short TXT victim.com | grep -i spf                 # SPF: mechanisms + the all qualifier (-all / ~all / ?all / +all)
dig +short TXT _dmarc.victim.com                        # DMARC: p=, sp=, pct=, adkim/aspf alignment mode
dig +short TXT selector._domainkey.victim.com           # DKIM: public key for a known selector (k=, p=, key length)
```

Lisez et routez :

- **Aucun enregistrement SPF, ou `+all` / `?all` / un `include:` envoyable** → le domaine d'enveloppe peut être usurpé directement. Allez à [SPF](spf.md).
- **DMARC manquant ou `p=none`** → l'en-tête `From:` n'est pas appliqué, les usurpations sont délivrées indépendamment de SPF/DKIM. Allez à [DMARC](dmarc.md).
- **`p=quarantine`/`p=reject` mais `sp=none`, `pct<100`, ou alignement relaxed** → les sous-domaines, une fraction du trafic, ou les identités sœurs se faufilent. Allez à [DMARC](dmarc.md).
- **Pas de DKIM, une clé courte, un tag `l=` de longueur de corps, ou un sélecteur obsolète** → la signature peut être forgée, étendue ou rejouée. Allez à [DKIM](dkim.md).
- **Les trois verrouillés** → repliez-vous sur des astuces qui ne nécessitent aucune faille d'enregistrement (usurpation de nom d'affichage, domaines ressemblants, incohérence d'en-tête). Allez à [Usurpation d'expéditeur](sender-spoofing.md).

Une fois la faille identifiée, [Usurpation d'expéditeur](sender-spoofing.md) est là où vous façonnez et envoyez réellement le message avec `swaks`.

## Pages

- **[SPF](spf.md)** : l'enregistrement `v=spf1`, ses mécanismes et qualificateurs, et les mauvaises configurations (`+all`, `~all` soft, `include:` trop large, permerror de limite de recherches) qui autorisent votre courrier.
- **[DKIM](dkim.md)** : l'en-tête `DKIM-Signature` et la clé publiée, et forger, étendre (`l=`) ou rejouer une signature valide.
- **[DMARC](dmarc.md)** : l'enregistrement `_dmarc`, l'alignement, et les failles de politique (`p=none`, `pct`, `sp` manquant, alignement relaxed, domaines cousins) qui laissent passer une usurpation du `From:`.
- **[Usurpation d'expéditeur](sender-spoofing.md)** : la page de récompense, façonner et envoyer le message usurpé, plus les astuces d'en-tête et de ressemblance qui fonctionnent même contre un domaine durci.

## Références

- [RFC 7489 (DMARC)](https://datatracker.ietf.org/doc/html/rfc7489)
- [RFC 7208 (SPF)](https://datatracker.ietf.org/doc/html/rfc7208)
- [RFC 6376 (DKIM)](https://datatracker.ietf.org/doc/html/rfc6376)
- [HackTricks : méthodologie de hameçonnage](https://book.hacktricks.wiki/en/generic-methodologies-and-resources/phishing-methodology/index.html)
