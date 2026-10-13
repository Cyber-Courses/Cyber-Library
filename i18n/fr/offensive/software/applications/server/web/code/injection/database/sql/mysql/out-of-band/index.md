---
order: 6
title: "Injection SQL out-of-band dans MySQL"
description: "Exfiltrer des données MySQL sur le réseau avec LOAD_FILE et des chemins UNC quand aucun canal in-band n'existe, et pourquoi l'out-of-band MySQL est largement réservé à Windows."
keywords:
  - injection out-of-band
  - exfiltration OOB
  - LOAD_FILE UNC
  - exfiltration DNS
  - fuite de hash NTLM
---

# Out-of-band

L'injection out-of-band (OOB) envoie des données vers un serveur contrôlé par l'attaquant au lieu de les lire dans la réponse. C'est l'option quand le résultat n'est pas reflété, que les erreurs sont supprimées et que le timing est trop lent ou peu fiable, à condition que la base puisse être amenée à atteindre le réseau.

La surface OOB de MySQL est étroite et principalement Windows. Il n'a pas de fonctions intégrées de résolveur HTTP ou DNS comme `UTL_HTTP` ou `UTL_INADDR` d'Oracle, donc le seul déclencheur général est `LOAD_FILE()` pointé vers un chemin UNC (`\\host\share\...`). Sous Windows, résoudre et se connecter à ce chemin fait effectuer à l'hôte une résolution DNS et une connexion SMB vers l'attaquant, ce qui à la fois exfiltre les données encodées dans le nom d'hôte et fait fuiter le hash NetNTLM du compte de service. Sur les builds Linux, `LOAD_FILE` ne suit pas les chemins UNC, donc ce canal ne s'applique généralement pas.

Chaque voie OOB ici nécessite le privilège `FILE` et un paramètre `secure_file_priv` qui ne bloque pas l'opération, le même verrou que la lecture et l'écriture de fichiers.

## Pages

- **[Exfiltration DNS](dns-exfiltration.md)** : encoder des données dans un nom d'hôte UNC pour qu'une résolution DNS les fasse fuiter.
- **[Capture UNC et NTLM](unc-ntlm.md)** : forcer l'authentification SMB pour capturer le hash du compte de service.

## Outils

- **sqlmap** : automatise l'exfiltration DNS out-of-band avec `--dns-domain`.
- **Burp Collaborator** : serveur de noms de journalisation pour l'exfiltration basée sur DNS.
- **Responder** : capturer l'authentification SMB que le chemin UNC déclenche.

## Références

- MySQL Reference Manual: LOAD_FILE, `secure_file_priv`, FILE privilege
- OWASP Testing Guide: Testing for SQL Injection
