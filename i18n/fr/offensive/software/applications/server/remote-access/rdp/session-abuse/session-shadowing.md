---
title: "Shadowing de session : observer une session RDP en direct"
description: "Le shadowing de session RDP permet à un compte privilégié de voir, et éventuellement de contrôler, la session active d'un autre utilisateur. Conçu pour l'assistance, il est détourné par un attaquant disposant de droits admin sur un hôte pour observer la session en direct d'une cible, capturant ce qu'elle tape et voit, et, le contrôle activé, pour interagir à sa place, le tout régi par une stratégie qui peut autoriser le shadowing sans consentement."
keywords:
  - session shadowing
  - rdp shadow
  - mstsc shadow
  - surveillance
  - terminal services
---

# Shadowing de session

Le shadowing est la fonctionnalité des services Bureau à distance qui permet à un utilisateur de voir ou de contrôler la session active d'un autre, conçue pour l'assistance du support. Un attaquant disposant de droits d'administration sur un serveur de terminaux la détourne en surveillance et interaction en direct : observer la session d'une cible capture tout ce qu'elle voit et tape (identifiants saisis, documents ouverts, commandes exécutées), et le contrôle activé, l'attaquant pilote la session en tant que cet utilisateur. Un paramètre de stratégie de groupe régit si le shadowing nécessite le consentement de l'utilisateur ; là où il est réglé pour autoriser la vue ou le contrôle sans consentement, la cible n'en a pas conscience.

```bash
# enumerate sessions, then shadow by ID
query user            # or qwinsta: find the target session ID
# shadow from an admin context (view or control per policy)
mstsc /shadow:<SESSION_ID> /v:<target> /control /noConsentPrompt
# the /noConsentPrompt and control behaviour depend on the Shadow policy on the host
```

## Notes d'exploitation

- Le comportement de consentement est défini par la stratégie « Définir les règles pour le contrôle à distance des sessions utilisateur des services Bureau à distance » ; là où elle permet la vue/le contrôle sans consentement, le shadowing est furtif, sinon l'utilisateur reçoit une invite.
- Le shadowing en lecture seule est une surveillance discrète, capturant identifiants et contenus sensibles pendant que l'utilisateur travaille ; le contrôle le transforme en action en tant que l'utilisateur dans sa session en direct.
- Comme le détournement, cela nécessite des droits d'administration existants sur l'hôte, c'est donc une technique post-accès sur les serveurs de terminaux et les hôtes de rebond ; elle diffère du [détournement de session](session-hijacking.md) en ce qu'elle observe/rejoint plutôt que de prendre le contrôle de la session.
- Utilisez-le pour récolter les identifiants qu'une cible tape et pour saisir des actions privilégiées en cours, puis pivotez avec ce qui est capturé.

## Références

- [Microsoft: RDS session remote control (shadowing)](https://learn.microsoft.com/en-us/windows-server/remote/remote-desktop-services/)
- [HackTricks: RDP shadowing](https://book.hacktricks.xyz/network-services-pentesting/pentesting-rdp)
