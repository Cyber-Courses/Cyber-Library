---
title: "Énumération des utilisateurs : valider les destinataires via SMTP"
description: "SMTP vous révèle quels comptes existent. VRFY et EXPN confirment et développent les destinataires directement, et RCPT TO après un MAIL FROM nul valide les adresses même quand VRFY et EXPN sont désactivés, les codes de réponse et le timing séparant les utilisateurs valides des inconnus. Les adresses validées alimentent la pulvérisation et l'usurpation."
keywords:
  - smtp user enumeration
  - VRFY
  - EXPN
  - RCPT TO
  - smtp-user-enum
---

# Énumération des utilisateurs

SMTP révèle l'espace de noms des destinataires parce que le serveur doit décider, par adresse, s'il acceptera du courrier pour elle. Trois verbes exposent cette décision. `VRFY <user>` demande au serveur de confirmer un utilisateur local : `250`/`252` signifie qu'il accepte l'adresse, `550` qu'il ne l'accepte pas. `EXPN <list>` développe une liste de diffusion ou un alias en ses membres. Les deux sont fréquemment désactivés, mais la décision d'acceptation se produit toujours lors d'une véritable transaction : après `MAIL FROM:<>` (un expéditeur nul, sur lequel les rebonds ne peuvent pas boucler), `RCPT TO:<user@domain>` renvoie `250` pour un destinataire livrable et `550` pour un inconnu. Ce chemin `RCPT TO` ne peut pas être désactivé sans refuser le courrier, c'est donc l'énumérateur fiable. Lorsque les codes sont rendus uniformes, le timing de réponse diffère souvent encore entre une véritable recherche de boîte aux lettres et un rejet instantané.

## Prérequis

- Atteignez le port 25 (ou 587). Lisez les capacités EHLO : `250-VRFY` et `250-EXPN` signifient que les verbes faciles sont disponibles ; leur absence vous renvoie simplement vers `RCPT TO`.
- Connaissez le domaine de messagerie. Tirez-le de la bannière, de l'enregistrement MX ou du PTR : `dig mx victim.com +short`.
- Ayez une liste de candidats (first.last, flast, noms d'employés issus de l'OSINT). L'énumération valide des candidats, elle ne les invente pas.

## Session guidée

```bash
nc mail.victim.com 25
```
```text
220 mail.victim.com ESMTP Postfix
EHLO attacker.test
250-mail.victim.com
250-VRFY
250 8BITMIME
VRFY root
250 2.1.5 root <root@victim.com>          # 250 = valid local user
VRFY nosuchuser
550 5.1.1 <nosuchuser>: Recipient address rejected: User unknown
# VRFY disabled? fall back to the transaction path:
MAIL FROM:<>
250 2.1.0 Ok
RCPT TO:<jsmith@victim.com>
250 2.1.5 Ok                               # 250 = jsmith exists / is deliverable
RCPT TO:<notreal@victim.com>
550 5.1.1 <notreal@victim.com>: Recipient address rejected: User unknown
```

Interprétez les codes : `250` accepté (valide), `550` rejeté (inconnu), `252` « ne peut VRFY mais tentera la délivrance » (ambigu : le serveur vous dit qu'il accepte tout et ne confirmera pas, de sorte que VRFY est inutile ici et vous devez juger par `RCPT TO` ou le timing à la place).

## L'automatiser

```bash
# RCPT mode: most reliable, works when VRFY/EXPN are off
smtp-user-enum -M RCPT -U users.txt -D victim.com -t mail.victim.com
# VRFY mode: faster where the verb is enabled
smtp-user-enum -M VRFY -U users.txt -t mail.victim.com
# Metasploit equivalent
msfconsole -q -x "use auxiliary/scanner/smtp/smtp_enum; set RHOSTS mail.victim.com; set USER_FILE users.txt; run; exit"
```

Lisez la sortie comme une séparation valide/invalide fondée sur la différence `250` contre `550`. `smtp-user-enum` affiche chaque nom d'utilisateur avec `exists` ou rien ; une exécution où tout « existe » signifie que le serveur accepte tout (style `252`), de sorte que le résultat est du bruit, pas une liste d'utilisateurs.

## Variantes et suites

- Exchange et Microsoft 365 ne se comportent pas comme un MTA classique ici : `RCPT TO` est souvent uniforme et VRFY a disparu. Énumérez-les plutôt via leurs points de terminaison web et le timing de connexion, voir [énumération Exchange](../../../exchange/enumeration.md) et [énumération de comptes Entra ID](../../../../../online/identity/entra-id/authentication/enumeration.md).
- Un domaine catch-all accepte chaque `RCPT TO`, mettant en échec la méthode. Confirmez-le en sondant d'abord une adresse aléatoire manifestement bidon ; si elle renvoie `250`, cessez de faire confiance à `RCPT TO`.
- Injectez chaque adresse validée directement dans l'[authentification](authentication.md) pour la pulvérisation de mots de passe, et dans toute charge utile d'[open relay](open-relay.md) ou de [SMTP smuggling](smtp-smuggling.md) comme de vrais destinataires internes.

## Outils

- [smtp-user-enum](https://github.com/pentestmonkey/smtp-user-enum)
- [Metasploit smtp_enum](https://github.com/rapid7/metasploit-framework/blob/master/modules/auxiliary/scanner/smtp/smtp_enum.rb)

## Références

- [RFC 5321 section 3.5 (VRFY et EXPN)](https://datatracker.ietf.org/doc/html/rfc5321#section-3.5)
- [pentestmonkey : smtp-user-enum](https://pentestmonkey.net/tools/user-enumeration/smtp-user-enum)
- [HackTricks : énumération de noms d'utilisateur SMTP](https://book.hacktricks.wiki/en/network-services-pentesting/pentesting-smtp/index.html)
