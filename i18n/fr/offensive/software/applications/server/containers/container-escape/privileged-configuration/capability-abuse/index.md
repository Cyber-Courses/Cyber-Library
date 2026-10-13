---
title: "Abus de capacité : des capacités Linux isolées qui brisent chacune le confinement"
description: "Comment des capacités Linux individuelles accordées à un conteneur offrent chacune une voie distincte vers l'hôte : CAP_SYS_ADMIN pour les montages et le release_agent de cgroup, CAP_SYS_MODULE pour le chargement de modules noyau, CAP_DAC_READ_SEARCH pour la lecture de fichiers arbitraires de l'hôte, CAP_SYS_PTRACE pour l'injection dans les processus de l'hôte, CAP_SYS_RAWIO et CAP_BPF pour l'accès direct à la mémoire du noyau, et CAP_NET_RAW pour l'interception réseau."
keywords:
  - capacités linux
  - cap_sys_admin
  - cap_sys_module
  - évasion de conteneur
  - décodage de capacités
---

# Abus de capacité

Un conteneur n'a pas besoin du flag privileged complet pour être évadable. Une seule capacité, ajoutée avec `--cap-add` ou un `securityContext.capabilities.add` Kubernetes, peut suffire, car chaque capacité correspond à un chemin de code du noyau qui n'était jamais censé être accessible à une charge de travail semi-fiable. L'ensemble de capacités par défaut de Docker est déjà une liste réduite ; les évasions dangereuses viennent de capacités ajoutées par-dessus, ou d'une image de base lancée avec `--cap-add=ALL`.

Décodez exactement ce que vous détenez avant de choisir une page :

```bash
grep CapEff /proc/self/status          # hex bitmask of effective capabilities
capsh --decode=00000000a80425fb        # => the Docker default set
capsh --print                          # human-readable current/bounding set
getpcaps $$                            # same, via libcap
```

Associez un bit de ce masque au nom de la capacité, puis allez à la page correspondante. Les ajouts à plus forte valeur, à peu près par ordre de directivité de l'évasion :

## Sous-sujets

- **[CAP_SYS_ADMIN](cap-sys-admin.md)** : mount, pivot_root, et l'évasion via le release_agent de cgroup.
- **[CAP_SYS_MODULE](cap-sys-module.md)** : charger un module noyau qui exécute du code sur l'hôte.
- **[CAP_DAC_READ_SEARCH](cap-dac-read-search.md)** : lire n'importe quel fichier de l'hôte en force brute sur les handles d'inode.
- **[CAP_DAC_OVERRIDE](cap-dac-override.md)** : contourner les vérifications de permissions de fichiers pour écrire des fichiers de l'hôte.
- **[CAP_SYS_PTRACE](cap-sys-ptrace.md)** : injecter du code dans les processus de l'hôte lorsque l'espace de noms PID est partagé.
- **[CAP_SYS_RAWIO](cap-sys-rawio.md)** : accès brut aux ports et à la mémoire physique pour patcher le noyau.
- **[CAP_BPF](cap-bpf.md)** : charger des programmes BPF qui lisent et écrivent la mémoire du noyau.
- **[CAP_NET_RAW](cap-net-raw.md)** : sockets raw pour le sniffing et l'usurpation sur le réseau du conteneur.

## Références

- [man 7 capabilities](https://man7.org/linux/man-pages/man7/capabilities.7.html)
- [HackTricks : Linux capabilities](https://book.hacktricks.xyz/linux-hardening/privilege-escalation/linux-capabilities)
- [Docker default capabilities (moby/oci defaults)](https://github.com/moby/moby/blob/master/oci/caps/defaults.go)
