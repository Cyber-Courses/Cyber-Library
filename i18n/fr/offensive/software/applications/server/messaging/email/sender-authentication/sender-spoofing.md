---
title: "Usurpation d'expéditeur : façonner un courrier qui semble provenir d'un domaine cible"
order: 4
description: "La page de récompense de l'authentification de l'expéditeur : façonner et envoyer un message usurpé avec swaks une fois que SPF, DKIM ou DMARC est manquant ou faible, et les astuces d'en-tête et de ressemblance qui ne nécessitent aucune faille d'enregistrement, usurpation de nom d'affichage, incohérence RFC5322.From contre RFC5321.MailFrom, redirection Reply-To, et domaines cousins punycode homoglyphes, délivrés via des open relays ou le SMTP smuggling."
keywords:
  - sender spoofing
  - email spoofing
  - display name spoofing
  - punycode domain
  - swaks
  - reply-to
---

# Usurpation d'expéditeur

C'est ici que la lecture de [SPF](spf.md), [DKIM](dkim.md) et [DMARC](dmarc.md) se transforme en un message délivré. Il existe deux familles de techniques. La première exploite une faille d'enregistrement afin que le domaine du `From:` usurpé passe directement l'authentification. La seconde ne nécessite aucune faille : elle abuse de la façon dont les clients de messagerie affichent un message, de sorte que même un domaine avec `p=reject` et un alignement strict peut être imité de façon convaincante pour l'humain qui le lit.

## Mécanisme : les deux identités From

Chaque message a deux identités d'expéditeur, et elles sont vérifiées et affichées différemment :

- **RFC5321.MailFrom** est le `MAIL FROM` d'enveloppe, le chemin de retour. SPF est vérifié contre son domaine. Le destinataire ne le voit presque jamais.
- **RFC5322.From** est l'en-tête `From:`. C'est ce que le client de messagerie affiche. L'alignement DMARC est jugé contre son domaine.

`swaks` définit les deux à partir de `--from` par défaut. Surcharger l'en-tête indépendamment (`--header "From: ..."`) est ce qui crée l'incohérence dont dépend l'usurpation : vous passez SPF sur un domaine que vous contrôlez tandis que le destinataire voit le domaine de la cible.

## Guidé : usurper avec une faille d'enregistrement

Lorsque le SPF de la cible autorise votre expéditeur, ou que DKIM est forgeable, ou que DMARC est `p=none`/absent, placez le domaine cible dans l'en-tête `From:` et envoyez. Utilisez un relais ou un MX qui acceptera la transaction (voir les voies de délivrance ci-dessous) :

```bash
swaks --to victim.employee@target.com \
      --from ceo@victim.com \
      --header "From: \"Jane Doe\" <ceo@victim.com>" \
      --header "Subject: Approval needed before 5pm" \
      --header "Date: $(date -R)" \
      --header "Message-ID: <$(openssl rand -hex 12)@victim.com>" \
      --body "Jane, can you action the attached before end of day?" \
      --server <relay-or-mx> --ehlo victim.com
# read the delivered Authentication-Results: dmarc=pass / dmarc=none confirms it survived
```

La voie du passage aligné dépend de la faille que vous avez utilisée : domaine d'enveloppe accepté par un SPF permissif, une signature DKIM forgée/rejouée qui s'aligne, ou simplement l'absence d'une politique DMARC appliquée. Confirmez en lisant l'`Authentication-Results:` dans le message délivré et en vérifiant s'il a atteint la boîte de réception.

## Guidé : usurpation de nom d'affichage (aucune faille d'enregistrement)

La plupart des clients de messagerie affichent le nom d'affichage en évidence et masquent ou tronquent l'adresse réelle. Un message entièrement authentifié pour un domaine jetable peut tout de même présenter un nom de confiance :

```bash
swaks --to victim.employee@target.com \
      --from attacker@sender-you-control.com \
      --header "From: \"Jane Doe (CEO, Victim Corp)\" <attacker@sender-you-control.com>" \
      --header "Subject: quick favor" \
      --body "Are you at your desk?" \
      --server mx.target.com
```

Ici SPF, DKIM et DMARC passent tous pour `sender-you-control.com` (que vous configurez correctement), de sorte que le message est propre selon chaque contrôle, et pourtant le client du destinataire affiche « Jane Doe (CEO, Victim Corp) ». Sur les clients mobiles qui n'affichent que le nom d'affichage, cela est souvent indiscernable de l'expéditeur réel.

## Guidé : redirection Reply-To

Associez l'une des techniques ci-dessus à un `Reply-To:` qui pointe vers une boîte de réception que vous contrôlez, afin qu'une réponse au message d'apparence de confiance vous parvienne même si le `From:` porte le nom de la cible :

```bash
swaks --to victim.employee@target.com \
      --from attacker@sender-you-control.com \
      --header "From: \"Jane Doe\" <ceo@victim.com>" \
      --header "Reply-To: Jane Doe <jane.doe.ceo@proton.me>" \
      --header "Subject: Re: vendor payment" \
      --body "Reply here, I am away from my usual mail." \
      --server mx.target.com
```

## Guidé : domaines ressemblants et homoglyphes

Les domaines cousins sont hors du périmètre DMARC de la cible parce qu'ils constituent un domaine organisationnel différent, de sorte qu'un domaine que vous enregistrez et authentifiez entièrement passe chaque contrôle. Deux variantes :

- **Typo/cousin** : enregistrez `victim-corp.com`, `victimcorp.com`, `victim-support.com`, publiez de vrais SPF/DKIM/DMARC, et envoyez du courrier authentifié. Tout est `pass` ; seul l'œil le détecte.
- **Homoglyphe IDN (punycode)** : enregistrez un nom utilisant des caractères Unicode qui s'affichent comme ceux de la cible. Générez le label `xn--` compatible ASCII et enregistrez-le :

```bash
# a Cyrillic 'а' (U+0430) in place of Latin 'a' renders identically in many fonts
python3 -c "d='vаlue.com'; print(d.encode('idna').decode())"   # -> xn--vlue-8kd.com (register this)
```

En envoyant depuis le domaine punycode, les clients qui affichent la forme Unicode montrent le nom de la cible dans l'adresse elle-même.

## Voies de délivrance

Le message usurpé a tout de même besoin d'un serveur qui l'acceptera et le transmettra :

- **Directement vers le MX destinataire** : fonctionne dès que le serveur entrant du destinataire accepte la transaction (`--server mx.target.com`). Convient aux usurpations par faille d'enregistrement et par nom d'affichage qui passent sur leur propre identité.
- **Open relay** : un serveur mal configuré qui transmet le courrier pour des expéditeurs arbitraires vous donne une sortie tierce et parfois une adresse IP source de confiance. Voir [open relay](../protocols/smtp/open-relay.md).
- **SMTP smuggling** : exploiter le désaccord sur la fin de données entre un relais sortant confiant et le serveur entrant pour injecter une seconde transaction usurpée qui hérite de l'identité authentifiée et alignée DMARC du relais. Voir [SMTP smuggling](../protocols/smtp/smtp-smuggling.md).

## Suites

Un message délivré et d'apparence de confiance est le point d'entrée, pas l'objectif. Les étapes suivantes courantes sont la compromission de messagerie d'entreprise (se faire passer pour un dirigeant ou un fournisseur afin de rediriger un virement ou de changer les coordonnées de paiement dans un fil existant) et le hameçonnage d'identifiants (un avis usurpé pointant vers une page de connexion clonée qui capture des identifiants ou un jeton MFA). Les deux reposent sur le `From:` auquel le destinataire fait confiance, ce que cette page produit ; le contenu, le contexte du fil et la page d'atterrissage sont le reste de l'opération.

## Outils

- [swaks](https://github.com/jetmore/swaks)
- [Gophish](https://github.com/gophish/gophish)
- [Référence des en-têtes de swaks](https://jetmore.org/john/code/swaks/)

## Références

- [RFC 5322 section 3.6.2 (champs d'origine : From et Reply-To)](https://datatracker.ietf.org/doc/html/rfc5322#section-3.6.2)
- [RFC 7489 (DMARC, alignement des identifiants et From)](https://datatracker.ietf.org/doc/html/rfc7489)
- [SEC Consult : SMTP smuggling, usurper des e-mails dans le monde entier](https://sec-consult.com/blog/detail/smtp-smuggling-spoofing-e-mails-worldwide/)
