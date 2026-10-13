---
title: "Tickets forgés : golden, silver, diamond et sapphire"
order: 4
description: "Forger des tickets Kerberos à partir de clés volées : golden tickets issus de la clé krbtgt, silver tickets issus d'une clé de service, et les variantes diamond et sapphire qui modifient de vrais tickets pour échapper à la détection basée sur le PAC."
keywords:
  - golden ticket
  - silver ticket
  - diamond ticket
  - sapphire ticket
  - krbtgt
---

# Tickets forgés

Kerberos fait confiance à un ticket parce qu'il peut être déchiffré avec la bonne clé et que son PAC est signé par le KDC. Si vous détenez la clé pertinente, vous pouvez **forger** un ticket avec l'identité et les privilèges de votre choix, car vous pouvez produire exactement ce que le vérificateur attend. La clé que vous détenez détermine ce que vous pouvez forger et l'ampleur de ce que vous obtenez.

**Filiation.** Le golden ticket a été introduit en 2014 et forge un TGT de toutes pièces en utilisant la clé krbtgt. Comme le contenu d'un ticket entièrement fabriqué peut paraître anormal aux yeux de la détection, les variantes diamond et sapphire modifient et re-signent plutôt un vrai ticket émis par le KDC (sapphire important un PAC authentique), en utilisant la même clé krbtgt tout en se fondant parmi les tickets légitimes.

## Golden ticket (la clé krbtgt)

La clé du compte `krbtgt` chiffre et signe chaque TGT du domaine. Avec elle (issue de [DCSync](../credentials/ntds-and-dcsync.md) ou d'un dump NTDS), vous forgez un TGT pour **n'importe quel** utilisateur avec **n'importe quelle** appartenance de groupe, valide jusqu'à ce que le mot de passe `krbtgt` change deux fois :

```bash
ticketer.py -nthash <krbtgt-hash> -domain-sid <sid> -domain example.local \
  -user-id 500 -groups 512,519,518,520 Administrator
export KRB5CCNAME=Administrator.ccache
```

Un golden ticket est à la fois une persistance à l'échelle du domaine, durable, et une impersonation, le tout dans un seul artefact. Forgez le SID d'un groupe privilégié (512 Domain Admins, 519 Enterprise Admins) pour agir en tant que ce groupe.

## Silver ticket (une clé de service)

Un silver ticket est un **ticket de service (TGS)** forgé, chiffré avec la clé d'un seul **compte de service** (un mot de passe de service cassé ou la clé d'un compte machine). Il ne contacte jamais le KDC, il est donc discret, mais il ne fonctionne que contre ce service unique :

```bash
ticketer.py -nthash <service-hash> -domain-sid <sid> -domain example.local \
  -spn cifs/server.example.local Administrator
```

La portée est le compromis : un silver ticket pour `cifs/host` accorde l'accès aux fichiers de cet hôte, un ticket `host/` accorde l'exécution de tâches planifiées/services, mais rien au-delà du SPN choisi.

## Diamond et sapphire (échapper aux vérifications du PAC)

Les golden et silver tickets construisent un PAC de toutes pièces, ce que les détections modernes signalent (combinaisons de groupes impossibles, champs manquants, horodatages incohérents). Les variantes diamond et sapphire partent plutôt d'un **vrai** ticket :

- **Diamond ticket** : demander un TGT légitime, le déchiffrer avec la clé `krbtgt`, modifier le PAC (ajouter des groupes), puis le rechiffrer. L'enveloppe du ticket est authentique, elle correspond donc à une vraie demande.
- **Sapphire ticket** : comme le diamond, mais on remplit le PAC avec le PAC d'un véritable utilisateur privilégié obtenu via S4U/U2U, de sorte que chaque champ est authentique plutôt que construit à la main.

```bash
Rubeus.exe diamond /krbkey:<krbtgt-aes> /user:user /password:pass /ticketuser:Administrator /ticketuserid:500 /groups:512
```

## Notes d'exploitation

- Préférez les clés **AES** lors de la forge : un golden ticket RC4 dans un domaine AES se remarque, tandis qu'une forge AES se fond dans la masse.
- Les silver tickets sont l'option la plus furtive pour une cible unique car ils ne touchent jamais un DC, idéale lorsque vous disposez d'une clé de service/machine et d'un objectif précis.
- La persistance par golden ticket survit à une seule réinitialisation de `krbtgt` ; les défenseurs doivent faire tourner `krbtgt` **deux fois** pour l'invalider, ce qui en fait une primitive de persistance de prédilection (voir la section Persistance).
- La clé d'un compte machine permet de forger des silver tickets pour les services de cet ordinateur et sous-tend l'abus de la [délégation](delegation/resource-based-constrained.md) basée sur les ressources.

## Outils

- **Impacket** (`ticketer.py`) : forge de golden et silver tickets à partir d'un hash ou d'une clé AES.
- **Rubeus** (`golden`, `silver`, `diamond`) : forge depuis l'hôte, y compris diamond.
- **Mimikatz** (`kerberos::golden`) : forge classique golden/silver.

## Références

- [Impacket ticketer (golden / silver)](https://github.com/fortra/impacket)
- [GhostPack Rubeus (golden / silver / diamond)](https://github.com/GhostPack/Rubeus)
- [Microsoft (MS-PAC): the Kerberos PAC](https://learn.microsoft.com/en-us/openspecs/windows_protocols/ms-pac/166d8064-c863-41e1-9c23-edaaa5f36962)
