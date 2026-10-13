---
title: "DMARC : failles de politique et d'alignement qui laissent passer une usurpation du From:"
order: 3
description: "Lire un enregistrement _dmarc et décider la voie d'usurpation selon sa politique : pas d'enregistrement, p=none, un pct sous 100, un sp= manquant ou faible qui laisse les sous-domaines ouverts, un alignement relaxed qui laisse passer une identité sœur, et les domaines cousins entièrement hors du périmètre DMARC. Inclut l'identification par dig et une usurpation de sous-domaine guidée."
keywords:
  - DMARC
  - DMARC alignment
  - p=none
  - sp subdomain policy
  - pct
  - relaxed alignment
---

# DMARC

DMARC est le seul des trois contrôles qui protège l'identité que le destinataire lit réellement, l'en-tête `From:`. Il fonctionne au-dessus de SPF et DKIM : un message passe DMARC lorsque SPF ou DKIM passe *et* que l'identité qui a passé *s'aligne* avec le domaine du `From:`. Il indique ensuite au destinataire quoi faire quand aucune ne s'aligne, et où envoyer les rapports. Comme DMARC est l'étape de liaison, son enregistrement de politique est la chose la plus importante à lire contre une cible : un domaine avec un SPF et un DKIM stricts mais une politique DMARC faible reste usurpable dans l'en-tête `From:`.

## Mécanisme

DMARC est un enregistrement TXT à `_dmarc.<domain>` :

```text
v=DMARC1; p=reject; sp=none; adkim=r; aspf=r; pct=100; rua=mailto:dmarc@victim.com
```

- `p=` est la politique pour le domaine organisationnel : `none` (ne rien faire, juste rapporter), `quarantine` (traiter comme suspect, généralement le dossier spam), `reject` (refuser au niveau SMTP).
- `sp=` est la politique pour les sous-domaines. Lorsque `sp=` est absent, les sous-domaines héritent de `p=`. Lorsqu'il est présent, il surcharge `p=` pour les seuls sous-domaines.
- `pct=` est le pourcentage de messages auxquels la politique est appliquée (par défaut 100). Le reste est traité selon la politique immédiatement inférieure.
- `adkim=`/`aspf=` règlent la rigueur de l'alignement : `r` relaxed (le défaut) correspond sur le domaine organisationnel, de sorte que `mail.victim.com` s'aligne avec `victim.com` ; `s` strict exige une correspondance de domaine exacte.

L'alignement est ce qui relie SPF/DKIM au `From:`. L'alignement SPF compare le domaine d'enveloppe `MAIL FROM` au domaine du `From:` ; l'alignement DKIM compare le `d=` de la signature au domaine du `From:`. Seul un passage aligné compte pour DMARC.

## Identifier

Récupérez l'enregistrement et lisez chaque tag :

```bash
dig +short TXT _dmarc.victim.com
# "v=DMARC1; p=reject; sp=none; adkim=r; aspf=r; pct=100; rua=mailto:dmarc@victim.com"
```

Décidez la voie à partir de ce que dit la politique :

- **Aucun enregistrement renvoyé** → le domaine ne publie aucun DMARC. Les destinataires n'ont rien qui lie l'en-tête `From:`, de sorte qu'un message non aligné est jugé sur le seul SPF/DKIM (et beaucoup le délivrent). Usurpez le `From:` directement.
- **`p=none`** → surveillance seule. Le domaine collecte des rapports mais demande aux destinataires de ne rien faire, de sorte que les usurpations non alignées sont délivrées. C'est la faille active la plus courante sur de vrais domaines.
- **`pct=` sous 100** → seule cette fraction du courrier reçoit `p` ; le reste retombe à la politique inférieure (pour `p=reject; pct=25`, les trois quarts des messages reçoivent au plus `quarantine`). Renvoyez la même usurpation à répétition ; une part atterrit sous le traitement plus doux.
- **`sp=none` (ou `sp=quarantine` alors que `p=reject`)** → le domaine organisationnel est appliqué mais pas les sous-domaines. Usurpez un sous-domaine (ci-dessous).
- **`adkim=r`/`aspf=r` (relaxed, le défaut)** → toute identité de sous-domaine s'aligne avec le parent. Si vous pouvez obtenir un passage SPF ou une signature DKIM pour *n'importe quel* sous-domaine (un sous-domaine marketing délégué à un fournisseur d'e-mails par lequel vous pouvez envoyer, un hôte oublié avec son propre SPF `+all`), un `From: anything@victim.com` s'aligne sous relaxed et passe.
- **`p=reject; sp=reject; adkim=s; aspf=s`** → entièrement verrouillé. Passez aux tactiques de ressemblance et de nom d'affichage sur [Usurpation d'expéditeur](sender-spoofing.md), que DMARC ne peut atteindre.

## Guidé : usurper un sous-domaine sous sp=none

Lorsque `sp=none` (ou que le parent est `p=none`), un sous-domaine de la cible n'est pas protégé, y compris les sous-domaines qui n'existent pas, parce que le destinataire remonte jusqu'à l'enregistrement `_dmarc` organisationnel et applique `sp`. Envoyez avec un sous-domaine à la fois dans l'enveloppe et dans l'en-tête `From:` :

```bash
# parent enforced, subdomains not: pick a plausible or non-existent subdomain
swaks --to target@recipient.com \
      --from alerts@notifications.victim.com \
      --header "From: \"IT Service Desk\" <helpdesk@noexist.victim.com>" \
      --header "Subject: Mailbox migration" \
      --body "..." \
      --server mx.recipient.com
```

Lisez l'`Authentication-Results:` délivré. Le message affiche toujours `dmarc=fail`, parce que ni SPF ni DKIM ne s'aligne avec le domaine du `From:` ; ce qui change est la disposition. La politique applicable du sous-domaine est `none` (définie explicitement par `sp=none`, ou héritée quand le parent est `p=none`), de sorte que l'action demandée au destinataire est de délivrer quand même. L'usurpation atterrit non pas parce que DMARC est passé mais parce que la politique pour ce sous-domaine demande de ne rien faire en cas d'échec. Le destinataire voit un sous-domaine `victim.com` dans le `From:`, et comme l'alignement relaxed est le défaut, cela se lit toujours comme l'organisation de confiance pour un humain.

## Guidé : profiter de l'alignement relaxed depuis une sœur

Sous l'alignement relaxed, vous n'avez pas besoin du domaine exact du `From:` pour passer SPF ou DKIM, seulement de son parent organisationnel. Si la cible délègue un sous-domaine à un expéditeur de masse (courant pour `email.victim.com`, `mktg.victim.com`) et que vous pouvez envoyer via cette plateforme, votre courrier obtient un passage SPF/DKIM aligné pour le sous-domaine, qui sous `adkim=r`/`aspf=r` s'aligne avec un `From: ceo@victim.com` :

```bash
# confirm a delegated subdomain with its own permissive SPF or a provider include you can use
dig +short TXT email.victim.com | grep -i spf     # e.g. "v=spf1 include:sendgrid.net -all"
# send through that provider so egress IP + envelope land inside the subdomain's authorized set,
# then set the visible From: to the bare parent domain; relaxed alignment makes it pass
```

Les domaines cousins ou ressemblants (`victim-support.com`, un homoglyphe punycode de `victim.com`) sont un domaine organisationnel entièrement différent, de sorte que DMARC ne s'y applique pas du tout ; ceux-ci relèvent des tactiques d'affichage sur [Usurpation d'expéditeur](sender-spoofing.md).

## Suites

La politique que vous lisez ici décide quelle faille antérieure vous avez réellement besoin : `p=none` ou aucun enregistrement signifie que toute usurpation du `From:` est délivrée sans aucun travail SPF/DKIM ; un `p` appliqué vous renvoie à [SPF](spf.md) ou [DKIM](dkim.md) pour obtenir un passage *aligné*. Une fois la voie claire, construisez et envoyez le message sur [Usurpation d'expéditeur](sender-spoofing.md).

## Outils

- [swaks](https://github.com/jetmore/swaks)
- [checkdmarc](https://github.com/domainaware/checkdmarc)

## Références

- [RFC 7489 (DMARC)](https://datatracker.ietf.org/doc/html/rfc7489)
- [RFC 7489 section 3.1 (alignement des identifiants)](https://datatracker.ietf.org/doc/html/rfc7489#section-3.1)
- [RFC 7489 section 6.3 (tags de l'enregistrement de politique)](https://datatracker.ietf.org/doc/html/rfc7489#section-6.3)
