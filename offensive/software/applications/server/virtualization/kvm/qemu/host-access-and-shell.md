---
title: "Host access and shell: execution on the QEMU/KVM host"
order: 1
description: "A KVM host is a Linux machine running QEMU processes under libvirt. Host access comes from a guest escape landing in a QEMU process, from libvirt or SSH access, or from host credentials. Because QEMU may run confined by seccomp and a dedicated user, the post-escape step is often a local privilege escalation to full host control over every VM."
keywords:
  - kvm host
  - qemu process
  - libvirt
  - seccomp
  - privilege escalation
---

# Host access and shell

A KVM host is an ordinary Linux server running one QEMU process per VM, typically managed by libvirt. Reaching the host happens three ways: a guest-to-host escape that lands code in a QEMU process, libvirt or SSH access with host credentials, or a host-side service compromise. What a QEMU escape gives depends on how the process is confined: libvirt commonly runs QEMU as a dedicated unprivileged user (`qemu`/`libvirt-qemu`), under a seccomp sandbox, and sometimes with SELinux/AppArmor (sVirt), so landing in QEMU is frequently a constrained foothold that still needs a local privilege escalation to reach full host control.

## From a QEMU foothold to the host

```bash
# determine the confinement of the QEMU process you landed in
id                                   # dedicated qemu/libvirt-qemu user?
grep Seccomp /proc/self/status       # seccomp filter active?
cat /proc/self/attr/current 2>/dev/null   # SELinux/AppArmor (sVirt) label
# enumerate what the qemu user can reach: other VMs' images, the libvirt socket
ls -l /var/lib/libvirt/images/ /var/run/libvirt/libvirt-sock 2>/dev/null
```

```bash
# with libvirt/SSH host access instead of an escape:
virsh list --all; virsh dumpxml <dom>          # control every VM
ssh root@<kvm-host>                            # full host if creds allow
```

## What host control gives

```bash
# every VM's disk is readable (see disk-and-snapshot theft), and the libvirt API
# controls all domains; persist on the host like any Linux server:
#  - a systemd unit, cron, or SSH key
#  - a libvirt hook (/etc/libvirt/hooks/qemu) runs on VM lifecycle events as root
```

## Exploitation notes

- Post-escape confinement is the key variable: a QEMU process running as a dedicated user under seccomp/sVirt means the escape yields that limited context, so plan a kernel or host-service local privilege escalation to reach root.
- libvirt or SSH host access is the simpler route when available and subsumes a guest escape: it controls every domain and reads every image directly.
- libvirt hooks (`/etc/libvirt/hooks/qemu`) execute as root on VM lifecycle events and are a clean host persistence location specific to KVM, alongside standard Linux mechanisms.
- Host control subsumes [Disk and snapshot theft](disk-and-snapshot-theft.md) and the [Management plane](management-plane.md) for every VM.

## References

- [libvirt security (sVirt) and sandboxing](https://libvirt.org/drvqemu.html#security)
- [QEMU security and seccomp](https://www.qemu.org/docs/master/system/security.html)
- [libvirt hooks](https://libvirt.org/hooks.html)
