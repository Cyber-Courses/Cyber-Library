---
title: "Force brute de mot de passe : attaques en ligne sur le login Telnet"
description: "Le login Telnet en clair peut être forcé par force brute, et les équipements qui l'exécutent n'appliquent souvent aucun verrouillage ni limitation de débit, ce qui rend la devinette praticable. Parce que la capture est plus facile, sniffer un login en clair l'emporte sur la devinette là où une position existe, mais la force brute avec des wordlists adaptées aux équipements reste efficace contre un Telnet exposé."
keywords:
  - force brute telnet
  - hydra
  - password spray
  - iot
  - clair
---

# Force brute de mot de passe

Le login en clair de Telnet peut être forcé par force brute, et l'équipement qui expose encore Telnet n'a fréquemment aucun verrouillage de compte ni limitation de débit, ce qui rend la devinette en ligne viable là où elle serait impraticable ailleurs. Les wordlists adaptées aux équipements (mots de passe par défaut et courants d'IoT/routeurs) sont efficaces, surtout combinées à un nom d'utilisateur dérivé des valeurs par défaut de l'équipement. Cela dit, parce que Telnet est en clair, si une position de capture existe, sniffer un vrai login ([sniffing de mot de passe](../traffic-interception/password-sniffing.md)) est plus discret et plus sûr que la devinette.

```bash
hydra -l admin -P passwords.txt telnet://<target> -t 4 -f
hydra -L users.txt -P iot-passwords.txt telnet://<target> -t 4
nmap -p23 --script telnet-brute --script-args userdb=u.txt,passdb=p.txt <target>
```

## Notes d'exploitation

- Adaptez la wordlist à l'équipement : les listes de valeurs par défaut et courantes d'IoT/routeurs surpassent les listes génériques de mots de passe humains contre du Telnet embarqué, et associez-les au nom d'utilisateur par défaut de l'équipement.
- De nombreux équipements Telnet n'ont pas de verrouillage, de sorte qu'une force brute qui verrouillerait un compte normal peut tourner librement ; gardez tout de même un nombre de threads modeste pour ne pas submerger les piles fragiles des équipements (certains plantent).
- Préférez la capture à la devinette lorsque vous êtes on-path : un sniff d'un login en clair donne l'identifiant exact sans tentative journalisée, voir [sniffing de mot de passe](../traffic-interception/password-sniffing.md).
- Une réussite est généralement administrative sur l'équipement et souvent réutilisée ; testez-la ailleurs.

## Outils

- [hydra](https://github.com/vanhauser-thc/thc-hydra)
- [nmap telnet-brute](https://nmap.org/nsedoc/scripts/telnet-brute.html)

## Références

- [HackTricks: Telnet brute force](https://book.hacktricks.xyz/network-services-pentesting/pentesting-telnet)
