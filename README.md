# zfs-build

GitHub Actions workflow to build OpenZFS `.deb` packages with pre-compiled kernel modules
for Ubuntu 24.04 (Noble), targeting kernel `6.8.0-134-generic`.

## Background

Ubuntu 24.04 ships OpenZFS 2.2.2, which contains a known use-after-free bug in the
`dbuf_hold_impl` read path (fixed in 2.2.3). The ZFS version is frozen for the LTS lifecycle
and cannot be upgraded via `apt`. This repo builds 2.2.10 — the latest 2.2.x release — as
`.deb` packages for manual installation.

Compiling on the live system is not possible: the compilation process reads ZFS source files
through the broken filesystem and reliably crashes the machine. All builds happen here on
GitHub Actions runners.

## What the Workflow Produces

- Standard `openzfs-*` `.deb` packages (userspace + DKMS source)
- `openzfs-zfs-modules-6.8.0-134-generic_2.2.10-1_amd64.deb` — pre-compiled kernel modules
  compressed as `.ko.zst` (required for Ubuntu 24.04's initramfs-tools to pick them up)

## Usage

1. Trigger the workflow manually via **Actions → Build OpenZFS 2.2.10 → Run workflow**
2. Download the artifact `zfs-2.2.10-noble-6.8.0-134`
3. Follow the installation instructions in the accompanying blog post

## Installation Notes

```bash
# Take a snapshot before installing
sudo zfs snapshot -r rpool/ROOT/ubuntu_zxbk71@before-zfs-upgrade

# Install (force-conflicts needed to replace Ubuntu's frozen packages)
sudo dpkg -i --force-overwrite --force-conflicts \
    openzfs-libnvpair3_2.2.10-1_amd64.deb \
    openzfs-libuutil3_2.2.10-1_amd64.deb \
    openzfs-libzfs4_2.2.10-1_amd64.deb \
    openzfs-libzpool5_2.2.10-1_amd64.deb \
    openzfs-libzfsbootenv1_2.2.10-1_amd64.deb \
    openzfs-zfsutils_2.2.10-1_amd64.deb \
    openzfs-zfs-zed_2.2.10-1_amd64.deb \
    openzfs-zfs-initramfs_2.2.10-1_all.deb \
    openzfs-zfs-modules-6.8.0-134-generic_2.2.10-1_amd64.deb

sudo depmod -a 6.8.0-134-generic
sudo update-initramfs -u -k 6.8.0-134-generic
sudo reboot
```

See the [blog post](https://blog.fdsboard.com/) for the full walkthrough including initramfs fixes.
