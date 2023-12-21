---
title: "Mounting NFS Share: Solving Bad Option Error"
date: 2023-12-17 11:11
categories: [Troubleshooting]
tags: [linux, nfs]
---

While doing the NFS section of HTB Academy's [Footprinting](https://academy.hackthebox.com/module/details/112) module, I encountered a problem when trying to mount the remote NFS share.

```console
$ sudo mount -t nfs 10.129.202.5:/var/nfs ./nfs-target/ -o nolock
mount: /home/slv3rfx/nfs-target: bad option; for several filesystems (e.g. nfs, cifs) you might need a /sbin/mount.<type> helper program.
```

When you encounter the above error message, you need to install the `nfs-common` package.[^1]

```console
$ sudo apt install nfs-common
```

After doing that, you should be able to mount the share without errors.

> If you are trying to mount a CIFS share, you need to install the `cifs-utils` package.
{: .prompt-tip}

[^1]: Source: [askubuntu.com](https://askubuntu.com/questions/525243/why-do-i-get-wrong-fs-type-bad-option-bad-superblock-error)

