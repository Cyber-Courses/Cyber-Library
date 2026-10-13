---
title: "Lecture mémoire kcore : extraire la mémoire du noyau de l'hôte via /proc/kcore"
order: 5
description: "/proc/kcore présente la mémoire du noyau en cours d'exécution sous forme de fichier core ELF. Un conteneur capable de lire le /proc/kcore de l'hôte lit la mémoire physique et la mémoire virtuelle du noyau, en extrayant des secrets, des clés et des structures d'identifiants. C'est une primitive de lecture puissante qui contourne l'isolation par espaces de noms, car la mémoire du noyau est une ressource unique à l'échelle de l'hôte."
keywords:
  - proc kcore
  - mémoire du noyau
  - extraction mémoire
  - divulgation d'informations
  - évasion de conteneur
---

# Lecture mémoire kcore

`/proc/kcore` est un fichier core ELF virtuel qui mappe la mémoire du noyau en cours d'exécution, y compris la RAM physique et l'espace d'adressage virtuel du noyau. Tout ce qui se trouve dans la mémoire du noyau, et une grande partie de ce qui se trouve en RAM, peut être lu à travers lui. Comme la mémoire du noyau est une ressource unique à l'échelle de l'hôte, non divisée par les espaces de noms, lire le `/proc/kcore` de l'hôte depuis l'intérieur d'un conteneur divulgue les secrets de l'hôte indépendamment de l'isolation du conteneur. C'est une primitive en lecture seule, mais d'une grande portée.

Confirmez l'accès et la taille :

```bash
ls -l /proc/kcore                              # present and readable
head -c 4 /proc/kcore | xxd                    # ELF magic (7f 45 4c 46) confirms the core format
```

## Extraction de la mémoire

Les en-têtes de programme ELF dans `/proc/kcore` décrivent quelles plages virtuelles correspondent à quels décalages. Des outils les analysent pour lire des adresses noyau précises ; une approche brutale scanne les segments lisibles à la recherche de secrets reconnaissables.

```bash
# Scan readable memory for key material and credentials
strings -n 12 /proc/kcore 2>/dev/null | grep -iE 'BEGIN (RSA|OPENSSH|EC) PRIVATE KEY' | head
strings /proc/kcore 2>/dev/null | grep -iE 'password|secret|token' | head
# Targeted: use a kernel symbol address from /proc/kallsyms, translate via the
# kcore ELF program headers, then read that offset
grep ' D ' /proc/kallsyms | grep -i init_task  # example symbol to anchor a read
```

Avec une table des symboles du noyau (`/proc/kallsyms` si elle est lisible) et les en-têtes de programme de kcore, des structures précises telles que les identifiants de processus ou les trousseaux de clés peuvent être localisées et lues avec précision, plutôt que de s'appuyer sur un balayage de chaînes.

## Notes d'exploitation

- Cela donne un accès en lecture, pas en écriture ; cela ne peut pas, à soi seul, exécuter du code. Sa valeur réside dans l'extraction de secrets (clés privées, jetons cloud et de service, `/etc/shadow` mis en cache en mémoire, clés de chiffrement) qui déverrouillent l'hôte ou d'autres systèmes.
- Les lectures précises nécessitent `/proc/kallsyms` avec de vraies adresses ; si `kptr_restrict` met les symboles à zéro, repliez-vous sur le balayage de signatures avec `strings` et des correspondances de motifs.
- L'accès implique généralement un conteneur privilégié ou un montage explicite du `/proc` de l'hôte ; un conteneur cloisonné par espaces de noms n'expose pas un `kcore` hôte exploitable. Pour un accès à la mémoire du noyau avec capacité d'écriture, voir [Périphériques de mémoire du noyau](../../privileged-configuration/device-access/kernel-memory-devices.md).

## Outils

- [volatility3 (analyse mémoire, peut analyser des sources de type kcore)](https://github.com/volatilityfoundation/volatility3)

## Références

- [man 5 proc : /proc/kcore](https://man7.org/linux/man-pages/man5/proc.5.html)
- [HackTricks : /proc/kcore](https://book.hacktricks.xyz/linux-hardening/privilege-escalation/docker-security/sensitive-mounts#proc-kcore)
