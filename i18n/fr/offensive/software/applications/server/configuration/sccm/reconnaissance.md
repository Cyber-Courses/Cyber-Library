---
title: "Reconnaissance SCCM : trouver les systèmes de site"
order: 1
description: "Découvrir un déploiement Configuration Manager depuis Active Directory et le réseau : le conteneur System Management, les management points, les distribution points et le serveur de site, à l'aide de sccmhunter et SharpSCCM avant tout accès privilégié."
keywords:
  - SCCM reconnaissance
  - System Management container
  - management point
  - sccmhunter
  - SharpSCCM
---

# Reconnaissance

Avant toute chose, vous devez savoir si SCCM est présent et quelle machine joue quel rôle : le **serveur de site** (le gros lot), les **management points** (où les clients récupèrent la politique), les **distribution points** (le contenu, y compris PXE) et la **base de données du site** (MSSQL). La plupart de cela est découvrable avec un seul identifiant de domaine, car SCCM se publie lui-même dans Active Directory.

## Depuis Active Directory

Lorsque le schéma est étendu, SCCM se publie dans le conteneur **System Management**, et les rôles sont identifiables par leur nommage et leurs SPN :

```bash
# sccmhunter: pull SCCM assets from the System Management container and fuzzy-match MECM/SCCM hosts
sccmhunter.py find -u user -p password -d example.local -dc-ip <dc>
sccmhunter.py show -all          # dump what was found (site servers, MPs, DPs)
```

```text
# Manually: the container and the MSSQL/management SPNs reveal the roles
LDAP: CN=System Management,CN=System,DC=example,DC=local
SPNs: MSSQLSvc/... on the site database, HTTP/... on management points
```

## Depuis le réseau

```bash
# SharpSCCM: identify the management point and site code from a client's perspective
SharpSCCM.exe local site-info
SharpSCCM.exe get site-info -d example.local   # queries a DC over LDAP
```

## Notes d'exploitation

- Le **serveur de site** est la cible de la [prise de contrôle](site-takeover.md) ; les **management points** sont l'endroit où les [identifiants de politique](credential-harvesting.md) sont récupérés ; les **distribution points** peuvent offrir du PXE.
- L'extension du schéma signifie que la reconnaissance ne nécessite souvent **qu'un utilisateur de domaine à faibles privilèges**, donc cela cartographie toute la hiérarchie avant que vous ne déteniez quoi que ce soit de privilégié.
- Notez le **site code** et si un central administration site se situe au-dessus du site primaire, car la portée de la prise de contrôle suit la hiérarchie.

## Outils

- **sccmhunter** (garrettfoster13) : découverte des actifs SCCM basée sur AD et profilage.
- **SharpSCCM** (Mayyhem) : énumération côté client et côté management point.
- **ldapsearch / BloodHound** : le conteneur System Management et les SPN directement.

## Références

- [SpecterOps: Misconfiguration Manager, RECON](https://github.com/subat0mik/Misconfiguration-Manager/tree/main/attack-techniques/RECON)
- [sccmhunter (garrettfoster13)](https://github.com/garrettfoster13/sccmhunter)
- [Truesec: common SCCM misconfigurations leading to privilege escalation](https://www.truesec.com/hub/blog/sccm-tier-killer)
