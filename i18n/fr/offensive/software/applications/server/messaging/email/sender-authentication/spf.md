---
title: "SPF : abuser des enregistrements Sender Policy Framework pour usurper l'expéditeur d'enveloppe"
order: 1
description: "Lire un enregistrement v=spf1 et ses mécanismes et qualificateurs pour trouver la faille qui autorise votre courrier : un enregistrement manquant, +all ou ?all, un include: trop large d'expéditeur partagé, un softfail permissif ~all, ou un permerror de dix recherches. Inclut l'identification par dig et une délivrance guidée avec swaks."
keywords:
  - SPF
  - Sender Policy Framework
  - v=spf1
  - all qualifier
  - SPF include
  - envelope spoofing
---

# SPF

SPF permet à un domaine de publier, dans un unique enregistrement DNS TXT, l'ensemble des adresses IP autorisées à envoyer du courrier en utilisant ce domaine dans l'enveloppe. Le destinataire prend le domaine du `MAIL FROM` SMTP (le chemin de retour, formellement RFC5321.From) et du nom `HELO`/`EHLO`, recherche l'enregistrement SPF de ce domaine, et vérifie si l'adresse IP de connexion correspond. Le fait décisif pour un attaquant : SPF ne regarde jamais l'en-tête `From:` que le destinataire voit, de sorte qu'un passage SPF à lui seul ne prouve rien sur l'expéditeur visible. SPF ne devient une barrière à l'usurpation que lorsque DMARC exige que l'identité SPF s'aligne avec le `From:`.

## Mécanisme

Un enregistrement SPF est un enregistrement TXT sur l'apex du domaine commençant par `v=spf1`, suivi de mécanismes évalués de gauche à droite, chacun portant un qualificateur :

```text
v=spf1 ip4:198.51.100.0/24 a mx include:_spf.google.com ~all
```

- **Les mécanismes** indiquent quels expéditeurs correspondent : `ip4:`/`ip6:` (plages littérales), `a` (les enregistrements A du domaine), `mx` (les hôtes MX du domaine), `include:` (importer l'enregistrement SPF d'un autre domaine et passer s'il passe), `all` (correspond à tout, toujours en dernier).
- **Les qualificateurs** préfixent un mécanisme et définissent le résultat en cas de correspondance : `+` pass (défaut si omis), `-` fail, `~` softfail, `?` neutral. Ainsi `-all` signifie « tout ce qui n'a pas correspondu au-dessus échoue », `~all` signifie « softfail », `?all` signifie « neutral », `+all` signifie « pass pour tout le monde ».

Le premier mécanisme correspondant l'emporte. Le qualificateur sur le `all` terminal décide donc du sort de tout expéditeur non explicitement listé.

## Identifier

Récupérez l'enregistrement et lisez le qualificateur de terminaison et les includes :

```bash
dig +short TXT victim.com | grep -i spf
# "v=spf1 include:_spf.google.com include:sendgrid.net ~all"
```

Points de décision à partir de ce que vous voyez :

- **Aucune ligne renvoyée** → pas de SPF publié. Le domaine d'enveloppe n'a aucune politique d'expéditeurs autorisés ; SPF ne peut pas échouer, et le fait que l'usurpation atterrisse dépend entièrement de DMARC/DKIM.
- **`+all` ou `?all`** → toute IP passe (`+all`) ou est neutre (`?all`) pour ce domaine. Vous pouvez envoyer au nom du domaine depuis votre propre hôte et SPF n'objectera pas.
- **`~all`** → softfail. Les expéditeurs non listés échouent « doucement », ce qu'une grande fraction de destinataires délivre tout de même (souvent directement dans la boîte de réception, parfois en spam). Vaut la peine d'envoyer pour confirmer.
- **`-all` avec un `include:` large** → le terminal est strict, mais un include d'expéditeur partagé (`sendgrid.net`, `spf.mandrillapp.com`, `_spf.google.com`, `servers.mcsv.net` pour Mailchimp) autorise *tout* le pool sortant du fournisseur. Si vous pouvez envoyer via ce fournisseur (un compte gratuit ou d'essai sur la même plateforme), votre courrier sort depuis une IP à l'intérieur de la plage incluse et SPF passe pour le domaine de la victime.
- **Nombreux termes `include:`/`a`/`mx`** → comptez les recherches DNS (voir ci-dessous) ; dépasser dix produit un permerror.

## Guidé : confirmer et délivrer

Comptez d'abord les recherches quand l'enregistrement est long. Chaque terme `include`, `a`, `mx`, `ptr` et `exists` coûte une recherche DNS, les includes imbriqués comptent de façon cumulative, et la limite est de dix. Au-delà de dix, c'est un `permerror`, que la plupart des destinataires traitent comme *pas un échec* (neutre), de sorte qu'un enregistrement surchargé désactive de fait l'application de SPF :

```bash
# a quick count of lookup-consuming terms across the record and its nested includes
dig +short TXT victim.com | tr ' ' '\n' | grep -E 'include:|^a$|^a:|^mx$|^mx:|ptr|exists:'
# pyspf resolves the full tree and reports permerror explicitly
python3 -c "import spf,sys; print(spf.check2(i='203.0.113.9', s='x@victim.com', h='mail.victim.com'))"
# ('permerror', 'Too many DNS lookups') => enforcement is effectively off
```

Lorsque le terminal est `~all`, `?all` ou `+all` (ou que l'enregistrement est en permerror), envoyez un message avec le domaine de la victime dans l'enveloppe et l'en-tête `From:` et montrez qu'il est accepté. `swaks` pilote la transaction SMTP complète pour que vous puissiez lire le code de réponse :

```bash
swaks --to target.user@recipient.com \
      --from notifications@victim.com \
      --helo victim.com \
      --header "Subject: Account notice" \
      --body "Test." \
      --server mx.recipient.com
# ... <-  MAIL FROM:<notifications@victim.com>
# ... -> 250 2.1.0 Ok
# ... <-  RCPT TO:<target.user@recipient.com>
# ... -> 250 2.1.5 Ok
# ... -> 250 2.0.0 Ok: queued           <- accepted; check the delivered Authentication-Results
```

Lisez l'en-tête `Authentication-Results:` dans le message délivré, et lisez le résultat SPF exactement, car un seul d'entre eux est un passage aligné :

- `spf=pass` : l'IP d'envoi a correspondu à un mécanisme (`ip4`/`ip6`/`a`/`mx`/`include`) ou l'enregistrement se termine par `+all`. C'est le seul passage aligné, et à lui seul il satisfait le DMARC basé sur SPF.
- `spf=softfail` (`~all`) et `spf=neutral` (`?all`) : pas un passage. Beaucoup de destinataires les délivrent tout de même, mais ce ne sont pas des résultats SPF alignés, de sorte qu'ils ne satisfont pas DMARC via SPF. L'usurpation atterrit alors seulement là où DMARC est absent ou réglé sur `p=none` (confirmez sur [DMARC](dmarc.md)).
- `spf=permerror` (plus de dix recherches DNS) et `spf=none` (pas d'enregistrement) : également pas un passage, avec la même réserve.

Ici `swaks --from` définit à la fois l'enveloppe et l'en-tête `From:` sur `victim.com`, de sorte que cela repose sur le seul SPF puis DMARC. Pour séparer les deux identités (un passage SPF sur un domaine que vous contrôlez tandis que l'en-tête `From:` usurpe la victime), voir [Usurpation d'expéditeur](sender-spoofing.md).

## Suites

Un passage SPF n'est que la moitié de l'histoire. Si le SPF de la victime vous laisse envoyer au nom de son domaine d'enveloppe, l'usurpation ne survit que lorsque DMARC n'exige pas aussi l'alignement DKIM ; lisez [DMARC](dmarc.md) ensuite pour confirmer la politique. Si SPF est strict mais qu'un `include:` partagé est en jeu, pivotez vers l'abus du pool de ce fournisseur. Dans les deux cas, le message est réellement construit et envoyé sur la page [Usurpation d'expéditeur](sender-spoofing.md).

## Outils

- [swaks](https://github.com/jetmore/swaks)
- [pyspf](https://github.com/sdgathman/pyspf)

## Références

- [RFC 7208 (Sender Policy Framework)](https://datatracker.ietf.org/doc/html/rfc7208)
- [RFC 7208 section 4.6.4 (limites de recherches DNS)](https://datatracker.ietf.org/doc/html/rfc7208#section-4.6.4)
- [HackTricks : usurpation d'e-mail (SPF)](https://book.hacktricks.wiki/en/generic-methodologies-and-resources/phishing-methodology/index.html)
