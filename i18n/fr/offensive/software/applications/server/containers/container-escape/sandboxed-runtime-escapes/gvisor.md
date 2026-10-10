---
title: "gVisor : s'évader du noyau en espace utilisateur et du proxy de fichiers"
description: "gVisor isole un conteneur derrière le Sentry, un noyau en espace utilisateur qui émule les appels système Linux, et le Gofer, un processus qui sert de proxy pour l'accès au système de fichiers. S'évader de gVisor revient à trouver un bug de sûreté mémoire ou de logique dans l'émulation d'appels système du Sentry, ou à abuser du protocole de proxy de fichiers du Gofer, afin de s'exécuter dans le contexte du Sentry ou de l'hôte plutôt qu'à l'intérieur de l'invité en bac à sable."
keywords:
  - gvisor
  - sentry
  - gofer
  - émulation d'appels système
  - évasion de bac à sable
---

# gVisor

gVisor ne laisse pas les appels système du conteneur atteindre le noyau de l'hôte. À la place, le Sentry, un noyau écrit en Go qui s'exécute en espace utilisateur, implémente lui-même l'interface d'appels système Linux, et un second processus, le Gofer, traite les requêtes du système de fichiers via le protocole 9P pour que le Sentry n'ouvre jamais directement les fichiers de l'hôte. La surface du noyau de l'hôte exposée au Sentry est délibérément minuscule et protégée par un filtre seccomp strict. S'évader signifie donc casser le Sentry ou le Gofer, et non le noyau de l'hôte : un bug dans l'émulation d'un appel système par le Sentry qui produit une corruption mémoire ou un accès hors limites dans l'espace d'adressage du Sentry, ou une faille dans le traitement du protocole Gofer qui permet au conteneur de référencer des fichiers de l'hôte en dehors de la racine prévue.

Confirmez gVisor et sondez la surface émulée :

```bash
cat /proc/version                              # gVisor prints a characteristic string
dmesg 2>/dev/null | grep -i gvisor
# emulation gaps: syscalls that behave differently or are unimplemented
strace -f /bin/true 2>&1 | grep -i 'not implemented\|ENOSYS' | head
```

## Surfaces d'évasion

- **Émulation d'appels système du Sentry** : le Sentry réimplémente des centaines d'appels système en Go. Une erreur de logique ou un défaut de sûreté mémoire dans un appel complexe (futex, splice, chemins de type io_uring, ou cas limites de l'émulation du système de fichiers et du réseau) peut corrompre l'état du Sentry. L'exécution de code dans le Sentry permet de s'évader de l'invité, après quoi l'interface hôte restreinte du Sentry constitue la prochaine barrière.
- **Proxy de fichiers Gofer** : le Gofer répond aux requêtes 9P pour le système de fichiers du conteneur. Une faille dans le traitement des chemins qui permet au conteneur de demander un fichier en dehors de sa racine mappée se transforme en accès au système de fichiers de l'hôte, puisque le Gofer s'exécute avec un accès à l'hôte plus étendu que l'invité en bac à sable.
- **Interface hôte après compromission du Sentry** : même avec l'exécution de code dans le Sentry, un filtre seccomp limite les appels système de l'hôte autorisés ; une évasion complète de l'hôte nécessite en outre un bug exploitable du noyau de l'hôte, atteignable à travers cette surface filtrée.

## Notes d'exploitation

- gVisor réduit et modifie la surface d'attaque plutôt que de la supprimer : la cible est le code Go du Sentry et le protocole Gofer 9P, de sorte que les techniques diffèrent nettement des évasions de conteneurs natives.
- L'incomplétude de l'émulation est un signal de départ utile ; les appels système qui renvoient des erreurs inhabituelles ou se comportent différemment d'un noyau natif marquent les zones où l'implémentation fait un travail non trivial et peut receler des bugs.
- Comme le Sentry est écrit en Go, langage à sûreté mémoire, la pure corruption mémoire est plus difficile ; les bugs de logique dans le mappage des chemins de fichiers et la gestion des ressources sont historiquement la classe la plus productive.

## Références

- [Modèle de sécurité de gVisor](https://gvisor.dev/docs/architecture_guide/security/)
- [Architecture de gVisor : Sentry et Gofer](https://gvisor.dev/docs/architecture_guide/)
- [Google Project Zero : recherche sur gVisor](https://googleprojectzero.blogspot.com/)
