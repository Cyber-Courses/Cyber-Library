---
title: "Stack overflow : débordements de buffer dans telnetd"
description: "telnetd a porté des débordements de buffer sur la pile dans sa gestion des options Telnet et des données de protocole, notamment les chemins de code de l'option de chiffrement et de telrcv, atteignables avant authentification. Sur les anciens builds Unix et embarqués sans protections de pile, une séquence d'options forgée déborde un buffer de taille fixe pour exécuter du code en tant que root."
keywords:
  - stack overflow
  - telnetd
  - telrcv
  - option de chiffrement
  - rce
---

# Stack overflow

L'analyse par le serveur Telnet des données de protocole et des options a produit des débordements de buffer sur la pile, où un buffer de taille fixe est rempli à partir d'une entrée contrôlée par l'attaquant sans borne. La classe la plus connue est dans la gestion de l'option de chiffrement et le chemin d'entrée `telrcv`, atteignable avant authentification, où une séquence d'options surdimensionnée ou forgée déborde un buffer de pile. Sur les anciens `telnetd` Unix et les builds embarqués qui exécutent encore Telnet, qui manquent couramment de canaris de pile, de piles non exécutables et d'ASLR, ce débordement est transformé en une exécution de code à distance fiable, et parce que `telnetd` s'exécute en tant que root, le résultat est root.

```bash
# fingerprint the telnetd build to match the specific overflow
nc <target> 23; nmap -p23 -sV <target>
# the exploit sends crafted Telnet option/negotiation data that overflows a stack
# buffer in the pre-auth parsing; on no-mitigation targets, overwrite the return
# address to shellcode/ROP. Build-specific offsets; match version to the advisory.
```

## Notes d'exploitation

- La propriété de grande valeur est la portée pré-authentification et le privilège root : l'analyse vulnérable s'exécute avant le login et `telnetd` est root, de sorte que le succès est un root non authentifié.
- Les systèmes Unix hérités et les appareils embarqués sont les cibles réalistes parce qu'ils manquent de protections modernes ; un telnetd moderne et protégé est bien plus difficile et ces bugs spécifiques sont corrigés.
- L'exploitation est spécifique au build (offsets, tailles de buffer) ; prenez l'empreinte de l'implémentation et de la version exactes et faites correspondre l'avis de sécurité.
- Les telnetd d'appareils (BusyBox et forks constructeur) ont leur propre historique de débordements ; traitez le Telnet embarqué comme un candidat de premier plan.

## Références

- [Avis de sécurité historiques sur les débordements de l'option de chiffrement de telnetd](https://www.cve.org/)
- [HackTricks: Telnet](https://book.hacktricks.xyz/network-services-pentesting/pentesting-telnet)
