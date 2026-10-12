---
title: "Détection de version : quelles versions de SNMP un agent accepte"
order: 1
description: "Un agent peut répondre à SNMPv1, v2c, v3 ou plusieurs à la fois, et la version détermine l'attaque. Sonder chaque version et observer lesquelles renvoient des données, et si v3 est proposée, révèle l'exposition : un équipement acceptant encore v1/v2c est attaquable par community string même si v3 est aussi configurée, ce qui constitue l'opportunité de downgrade courante."
keywords:
  - snmp version detection
  - v1 v2c v3
  - downgrade
  - nmap snmp
  - probe
---

# Détection de version

Déterminer quelles versions de SNMP un agent accepte est l'étape préalable, car les versions ont des attaques différentes et les agents en acceptent fréquemment plus d'une. Sonder avec chaque version et voir lesquelles renvoient des données vous indique l'exposition. Un équipement qui répond à v1 ou v2c est attaquable via la community string en clair, que v3 soit aussi configurée ou non, ce qui est la constatation courante et précieuse : les administrateurs activent v3 mais laissent v1/v2c pour l'outillage hérité, donc le chemin faible subsiste. À l'inverse, un équipement qui ne répond qu'à v3 exige les attaques propres à v3.

```bash
# probe each version against a known OID
snmpget -v1  -c public <target> 1.3.6.1.2.1.1.1.0 2>/dev/null && echo "v1 ok"
snmpget -v2c -c public <target> 1.3.6.1.2.1.1.1.0 2>/dev/null && echo "v2c ok"
# v3 presence: engine discovery returns an engine ID even without credentials
snmpget -v3 -l noAuthNoPriv -u probe <target> 1.3.6.1.2.1.1.1.0 2>&1 | grep -i engineID
# nmap confirms the agent and attempts version/strings
nmap -sU -p161 -sV --script snmp-info <target>
```

## Notes d'exploitation

- La constatation clé est un équipement qui accepte encore v1/v2c : c'est le chemin faible, et sa présence aux côtés de v3 est une opportunité de downgrade, utilisez les attaques par community string et ignorez v3.
- Un équipement ne répondant qu'à v3 impose les attaques v3 (énumération des noms d'utilisateurs, cassage hors ligne, tentative de downgrade) ; confirmez par l'absence de toute réponse v1/v2c et la présence d'un engine ID lors de la découverte v3.
- Les agents sont en UDP et muets face à de mauvais identifiants, donc distinguez « mauvaise chaîne » de « version non prise en charge » en testant une chaîne connue comme valide par version lorsque vous en avez une.
- La version détectée oriente vers [SNMPv1 et v2c en clair](snmpv1-and-v2c-cleartext.md) ou vers [Attaques SNMPv3](snmpv3-attacks.md).

## Références

- [nmap snmp-info](https://nmap.org/nsedoc/scripts/snmp-info.html)
- [HackTricks: SNMP](https://book.hacktricks.xyz/network-services-pentesting/pentesting-snmp)
