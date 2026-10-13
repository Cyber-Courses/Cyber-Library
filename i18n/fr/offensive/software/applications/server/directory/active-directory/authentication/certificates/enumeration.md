---
title: "Énumération AD CS : trouver les CA, les modèles et les faiblesses"
description: "Énumérer Active Directory Certificate Services depuis un compte de domaine : localiser les CA d'entreprise, les modèles de certificat publiés, les droits d'inscription, et les mauvaises configurations (conditions ESC) qui rendent les certificats exploitables."
keywords:
  - certipy find
  - certificate template
  - enterprise CA
  - pKIEnrollmentService
  - ESC
---

# Énumération AD CS

La configuration d'AD CS réside dans la **partition de configuration** de l'annuaire, lisible par tout utilisateur authentifié. Cela signifie qu'un seul compte de domaine suffit à inventorier chaque CA, chaque modèle publié, qui peut s'inscrire, et quels modèles ou CA sont mal configurés, avant de toucher à quoi que ce soit. L'énumération détermine quelle escalade (s'il y en a une) est disponible, c'est donc toujours la première étape.

## Ce qu'il faut collecter

- **CA d'entreprise** : les objets `pKIEnrollmentService` sous `CN=Enrollment Services`, donnant le nom de chaque CA, son hôte DNS et les modèles qu'elle publie.
- **Modèles de certificat** : les objets `pKICertificateTemplate`, avec leurs EKU, leurs drapeaux d'inscription, et l'ACL qui indique qui peut s'inscrire.
- **L'objet NTAuthCertificates** : quelles CA sont de confiance pour l'authentification dans la forêt.
- **Droits d'inscription et ACL d'objets** : qui peut s'inscrire sur chaque modèle, et qui peut *écrire* sur les modèles, les CA ou les conteneurs PKI.

## Lancer l'énumération

```bash
# Certipy: collect everything and flag vulnerable configurations
certipy find -u user@example.local -p pass -dc-ip <dc> -stdout -vulnerable

# NetExec from Linux: -M adcs lists the enrolment servers (CAs); pass SERVER to
# enumerate that CA's templates; certipy-find runs Certipy's triage inline
nxc ldap <dc> -u user -p pass -M adcs
nxc ldap <dc> -u user -p pass -M adcs -o SERVER=<CA-CN>
nxc ldap <dc> -u user -p pass -M certipy-find

# BloodHound (with AD CS collection) graphs CA/template relationships and ESC paths
# Certutil, on a domain-joined host
certutil -config - -ping
certutil -template
```

`certipy find -vulnerable` est l'outil de prédilection : il lit les modèles et les paramètres de la CA et étiquette chaque condition **ESC** applicable, vous pouvez donc passer directement à la technique correspondante.

## Lire les résultats

Les conditions qui rendent un modèle ou une CA exploitable, et la page qui couvre chacune :

- Sujet fourni par l'inscrit + un EKU d'authentification + inscription peu privilégiée : **[ESC1](vulnerable-templates.md)**.
- Any Purpose / aucun EKU, ou modèles Enrollment Agent : **[ESC2/ESC3](vulnerable-templates.md)**.
- Extension de sécurité manquante, ou mappage de compte faible : **[ESC9/ESC10/ESC14](certificate-mapping.md)**.
- Objet modèle/CA/PKI accessible en écriture : **[ESC4/ESC5/ESC7](access-control.md)**.
- Drapeau d'attribut SAN de la CA, ou liaison SID désactivée : **[ESC6/ESC16](ca-configuration.md)**.
- Inscription web ou RPC accessible pour relais : **[ESC8/ESC11](relay-to-adcs.md)**.

## Notes d'exploitation

- L'énumération est peu privilégiée et peu bruyante (lectures d'annuaire plus un ping de la CA), lancez-la donc tôt avec n'importe quel identifiant valide.
- Un modèle n'est exploitable que si vous détenez aussi des **droits d'inscription** sur celui-ci, directement ou via un groupe ; recoupez l'ACL du modèle avec vos appartenances de groupe.
- Le nom DNS de l'hôte de la CA et ses points de terminaison d'inscription (inscription web, CES) alimentent les techniques de relais, notez-les donc même lorsqu'un chemin par modèle semble plus simple.

## Outils

- **Certipy** (`find -vulnerable`) : l'outil principal d'énumération et de triage AD CS.
- **NetExec (`nxc`) `-M adcs` / `-M certipy-find`** : découverte des CA et des modèles, et triage Certipy, via LDAP depuis Linux.
- **BloodHound** (collecte AD CS) : graphes des relations CA/modèle/inscription.
- **certutil / PSPKIAudit** : énumération native et PowerShell.

## Références

- [Certipy wiki: privilege escalation (ESC triage)](https://github.com/ly4k/Certipy/wiki/06-%E2%80%90-Privilege-Escalation)
- [NetExec adcs module source](https://github.com/Pennyw0rth/NetExec/blob/main/nxc/modules/adcs.py)
- [SpecterOps: Certified Pre-Owned](https://specterops.io/blog/2021/06/17/certified-pre-owned/)
