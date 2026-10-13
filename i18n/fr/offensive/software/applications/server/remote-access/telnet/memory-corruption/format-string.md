---
title: "Format string : failles de chaîne de format dans telnetd"
description: "telnetd et le code associé ont porté des vulnérabilités de chaîne de format où une entrée contrôlée par l'attaquant, telle que des données d'environnement ou de terminal transmises via Telnet, atteint une fonction de la famille printf en tant qu'argument de format. Cela donne une lecture et une écriture en mémoire, et sur les builds vulnérables une exécution de code à distance, généralement en tant que root."
keywords:
  - format string
  - telnetd
  - printf
  - écriture mémoire
  - rce
---

# Format string

Les bugs de chaîne de format surviennent lorsqu'une donnée contrôlée par l'attaquant est passée comme argument de format à une fonction de la famille `printf`, permettant à l'attaquant d'utiliser des spécificateurs de format (`%x`, `%n`) pour lire et écrire en mémoire. Telnet a exposé cette classe là où une donnée fournie par le client, variables d'environnement, type de terminal ou valeurs d'option transmises par `telnetd` vers du logging ou du traitement, atteint une fonction de formatage sans être assainie. Le spécificateur `%n` offre une écriture arbitraire, qui sur un build vulnérable est escaladée vers un contrôle du flux d'exécution et une exécution de code à distance, là encore généralement en tant que root étant donné le privilège de `telnetd`.

```bash
# fingerprint to match the specific format-string flaw
nc <target> 23; nmap -p23 -sV <target>
# the exploit supplies format specifiers in the vulnerable field (e.g. an option/
# environment value telnetd logs or processes with printf-family code):
#   %x...%x to leak stack/memory, %n to write; chain the write to hijack control flow.
# offsets are build-specific; match version to the advisory.
```

## Notes d'exploitation

- Les failles de chaîne de format donnent à la fois une lecture (fuite, utile pour vaincre l'ASLR là où il est présent) et une écriture (`%n`), ce sont donc des primitives flexibles capables d'amorcer elles-mêmes un exploit sur des cibles protégées comme non protégées.
- L'entrée vulnérable est le champ contrôlé par le client qui atteint une fonction de formatage ; sur Telnet, cela a inclus des données d'environnement/terminal transmises et des chemins de logging.
- Comme pour les débordements, les telnetd hérités et embarqués s'exécutant en tant que root sont les cibles réalistes, et l'exploitation est spécifique au build.
- Là où aucun bug de corruption mémoire ne s'applique, les surfaces en clair et d'auth faible de Telnet demeurent ; cette classe est le chemin de RCE directe lorsque le build est vulnérable.

## Références

- [Historic telnetd format-string advisories](https://www.cve.org/)
- [HackTricks: Telnet](https://book.hacktricks.xyz/network-services-pentesting/pentesting-telnet)
