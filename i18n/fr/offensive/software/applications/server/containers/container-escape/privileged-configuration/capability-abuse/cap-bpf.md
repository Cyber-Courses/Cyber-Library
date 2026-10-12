---
title: "CAP_BPF : lire et écrire la mémoire du noyau via des programmes BPF"
order: 8
description: "CAP_BPF permet de charger des programmes BPF et de créer des maps. À elle seule, ou combinée à CAP_PERFMON ou CAP_SYS_ADMIN, elle offre une voie pour lire une mémoire noyau arbitraire via bpf_probe_read et, là où les types de programmes de tracing et les helpers sont accessibles, pour écraser des données du noyau, puisque les programmes BPF s'exécutent dans le contexte du noyau partagé par tous les espaces de noms."
keywords:
  - cap_bpf
  - bpf syscall
  - bpf_probe_read
  - mémoire du noyau
  - évasion de conteneur
---

# CAP_BPF

`CAP_BPF` a été séparée de `CAP_SYS_ADMIN` afin de permettre le chargement de programmes BPF et la création de maps BPF sans la capacité d'administration complète. Les programmes BPF s'exécutent à l'intérieur du noyau après avoir passé le vérificateur, et ils partagent l'unique noyau hôte entre tous les espaces de noms. Avec `CAP_BPF`, et selon les types de programmes et les helpers autorisés par le noyau, un attaquant peut lire la mémoire du noyau et, avec des programmes de tracing accompagnés de helpers tels que `bpf_probe_write_user` ou de primitives basées sur les maps, influencer la mémoire du noyau et celle de l'espace utilisateur, s'évadant ainsi de l'isolation du conteneur.

Confirmez la capacité et les types de programmes accessibles :

```bash
capsh --print | grep -o -E 'cap_bpf|cap_perfmon|cap_sys_admin'
grep CapEff /proc/self/status          # cap_bpf is bit 39
bpftool feature probe 2>/dev/null | grep -i 'program_type .* available' | head
```

## Voie : lire la mémoire du noyau

Un programme BPF de tracing (par exemple `BPF_PROG_TYPE_KPROBE` ou `BPF_PROG_TYPE_TRACEPOINT`, qui nécessitent habituellement aussi `CAP_PERFMON`) peut appeler `bpf_probe_read_kernel` pour copier des adresses noyau arbitraires dans une map que l'attaquant lit ensuite depuis l'espace utilisateur. Cela fuit des identifiants, des pointeurs permettant de vaincre KASLR, et le contenu de structures du noyau.

```c
// sketch of a kprobe program body (loaded via bpf() BPF_PROG_LOAD)
// reads N bytes at a target kernel address into a perf/array map
bpf_probe_read_kernel(&out, sizeof(out), (void *)target_kaddr);
bpf_map_update_elem(&results, &key, &out, BPF_ANY);
```

```bash
# user side: pin the map, poll it for leaked kernel data
bpftool map dump name results
```

## Voie : primitives d'écriture

Lorsque `bpf_probe_write_user` est disponible pour le type de programme chargé, un programme de tracing qui se déclenche dans le contexte d'une tâche cible peut écraser la mémoire utilisateur de cette tâche. Combiné à un helper qui atteint une structure d'identifiants, cela est escaladé jusqu'à l'écrasement de champs UID ou à la désactivation d'une vérification. Les helpers réellement accessibles dépendent de la version du noyau et de la combinaison de capacités ; `CAP_BPF` avec `CAP_PERFMON` déverrouille les types de programmes de tracing qui rendent ces helpers utilisables.

## Notes d'exploitation

- Le vérificateur BPF et les listes d'autorisation de helpers par type de programme sont le véritable verrou. `CAP_BPF` seule vous limite à un sous-ensemble ; l'associer à `CAP_PERFMON` (tracing) ou à `CAP_SYS_ADMIN` élargit grandement ce qui peut être chargé. Sondez avec `bpftool feature probe`.
- `kernel.unprivileged_bpf_disabled` ne s'applique pas ici puisque vous détenez la capacité, mais `CONFIG_BPF_UNPRIV_DEFAULT_OFF` et une surface `bpf()` verrouillée peuvent tout de même bloquer certains types de programmes.
- Historiquement, plusieurs bugs du vérificateur BPF permettaient une lecture/écriture noyau arbitraire depuis des programmes chargeables ; lorsque le noyau hôte n'est pas corrigé, un programme de contournement du vérificateur est une évasion directe et pas seulement une fuite.

## Outils

- [bpftool (inspection et chargement BPF du noyau)](https://github.com/libbpf/bpftool)

## Références

- [man 2 bpf](https://man7.org/linux/man-pages/man2/bpf.2.html)
- [Kernel docs : BPF design and capabilities](https://docs.kernel.org/bpf/index.html)
- [HackTricks : CAP_BPF](https://book.hacktricks.xyz/linux-hardening/privilege-escalation/linux-capabilities)
