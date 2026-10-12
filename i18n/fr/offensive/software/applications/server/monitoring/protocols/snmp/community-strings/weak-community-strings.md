---
title: "Chaînes de communauté faibles : valeurs prévisibles et dérivées de l'organisation"
description: "Là où les valeurs par défaut sont modifiées, la chaîne de communauté de remplacement est fréquemment faible : un mot court, le nom de l'entreprise ou du site, le nom d'hôte, un motif saison-et-année, ou une variation triviale de public/private. Elles se devinent à partir de l'organisation et de l'équipement, de sorte qu'une petite liste ciblée récupère souvent la chaîne sans force brute complète."
keywords:
  - weak community string
  - predictable
  - organization name
  - hostname
  - guessable
---

# Chaînes de communauté faibles

Lorsqu'un administrateur modifie effectivement la chaîne de communauté, le remplacement n'est généralement pas robuste, car les opérateurs la traitent comme une étiquette plutôt que comme un secret et elle doit être configurée à l'identique sur de nombreux équipements. Le résultat est prévisible : un mot court du dictionnaire, le nom de l'entreprise ou du site, le nom d'hôte ou le modèle de l'équipement, un motif saison-et-année (les mêmes que ceux utilisés pour les mots de passe), ou une variation triviale comme `public1`, `publicRO` ou `snmp`. Elles se devinent à partir du contexte, de sorte que construire une petite liste ciblée à partir du nom de l'organisation, du nom d'hôte (visible dans `sysName` si une chaîne fonctionne) et des motifs courants récupère fréquemment la chaîne avec bien moins de tentatives qu'une force brute complète.

```bash
# build a context list from the org and device, then test it
printf '%s\n' public private snmp "$ORG" "${ORG}snmp" "${ORG}123" \
  "$HOSTNAME" public1 publicRO publicRW community monitor Spring2025 \
  > comm.txt
onesixtyone -c comm.txt <target>
# or iterate with snmpget
while read c; do snmpget -v2c -c "$c" -t1 -r0 <target> 1.3.6.1.2.1.1.1.0 2>/dev/null \
  && echo "WORKS: $c"; done < comm.txt
```

## Notes d'exploitation

- Dérivez la liste du contexte : noms de l'organisation et des produits, nom d'hôte et emplacement (si une chaîne fonctionne déjà, lisez d'abord `sysName`/`sysLocation`), et les motifs de mots de passe que l'organisation utilise ailleurs ; cela bat une liste de mots générique en vitesse et en discrétion.
- La même chaîne de communauté est généralement réutilisée sur tout le parc (elle est configurée dans un modèle), de sorte qu'une chaîne récupérée déverrouille souvent tous les équipements de cette classe ; testez-la largement.
- Les chaînes faibles se situent entre les [valeurs par défaut](default-community-strings.md) et la [force brute](brute-force.md) complète ; essayez la liste ciblée avant la grande liste de mots.
- SNMP n'a pas de verrouillage, de sorte que les tentatives ne sont contraintes que par les délais d'expiration UDP ; gardez la liste concise pour rester rapide et discret.

## Références

- [HackTricks : SNMP](https://book.hacktricks.xyz/network-services-pentesting/pentesting-snmp)
- [SecLists SNMP strings](https://github.com/danielmiessler/SecLists)
