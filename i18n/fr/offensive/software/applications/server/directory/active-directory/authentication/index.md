---
title: "Authentification : la surface d'attaque des identifiants et de l'identité Active Directory"
order: 1
description: "Le cœur des attaques Active Directory : obtenir et détourner le matériel d'authentification sous toutes ses formes, mots de passe et empreintes NTLM, tickets Kerberos et certificats, via l'extraction, le cassage, le roasting, le relais et la forgerie."
keywords:
  - credential access
  - NTLM
  - kerberos
  - AD CS
  - pass the hash
---

# Authentification

C'est le cœur des attaques Active Directory. Les mots de passe, les empreintes NTLM, les tickets Kerberos et les certificats sont tous la même chose vue sous différents angles : du **matériel d'authentification** qui prouve au domaine qui vous êtes. La boucle centrale d'un attaquant consiste à en obtenir une forme, à convertir entre les formes, et à le réutiliser pour s'authentifier en tant que principal plus privilégié. C'est pourquoi on les traite ensemble plutôt que séparément : une empreinte cassée devient un mot de passe, un mot de passe devient un ticket Kerberos, un ticket ou un certificat devient de l'exécution de code, et l'exécution de code extrait davantage d'empreintes.

## Les formes du matériel d'authentification

- **Secrets** (mots de passe et leurs empreintes NTLM/Kerberos) : obtenus en les extrayant de la mémoire ou des bases de données d'un hôte, en les cassant hors ligne, ou en les devinant par spraying.
- **Authentification NTLM** : l'échange challenge-réponse, qui peut être capturé, relayé ou rejoué avec la seule empreinte (pass-the-hash).
- **Tickets Kerberos** : demandés, roastés pour leur clé de chiffrement, forgés de toutes pièces, ou passés pour usurper l'identité d'un principal.
- **Certificats** : émis par AD Certificate Services et utilisables comme matériel d'authentification (PKINIT), forgés ou volés via des mauvaises configurations de templates.

## Comment cette section est organisée

- **[Identifiants](credentials/index.md)** : extraire les secrets des hôtes et de l'annuaire, les casser, et les deviner par spraying, avec l'énumération des utilisateurs et des politiques qui cible le spraying.
- **[NTLM](ntlm/index.md)** : capturer, relayer et rejouer l'authentification NTLM (pass-the-hash).
- **[Kerberos](kerberos/index.md)** : roasting, forgerie de tickets, abus de délégation et réutilisation de tickets.
- **[Certificats](certificates/index.md)** : détourner AD Certificate Services pour l'authentification et l'élévation.

## Références

- The Hacker Recipes: Active Directory movement (credentials, NTLM, Kerberos)
- SpecterOps: Certified Pre-Owned (AD CS)
