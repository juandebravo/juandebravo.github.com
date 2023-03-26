---
layout: post
title: "How to mount an external volume with write permissions"
categories: [osx, usb]
---

I am always struggling to figure out how to fix “read-only permissions” when plugging an external USB into your Mac. If you are on my team, this has worked for me:

1. Get the list of mounted volumes:

```bash
diskutil list
```

You will get a list of volumes, both internal (your hard disk(s)) and external (your USB).

2. Identify the external one with read-only permissions (e.g. `/dev/disk4`) and unmount it:

```bash
sudo diskutil unmountDisk /dev/disk4
```

In my case it's always `/dev/disk4` :man_shrugging:

3. Create a folder where the USB will be mounted:

```bash
sudo mkdir /Volumes/EXTERNAL_USB
```

4. Mount the USB again with write permissions:

```bash
sudo mount -w -t exfat /dev/disk4s1 /Volumes/EXTERNAL_USB
```

where:

- `-w` stands for write permissions.
- `-t exfat` is the external type. It should match the format you used when formatting your USB (e.g. `exfat`, `nfs`, `msdos`, etc).
- `/dev/disk4s1` is the disk identifier + partition to mount.
- `/Volumes/EXTERNAL_USB` is the folder you created in step 3.

The tricky thing to remember is the external type you should use. Valid options can be found by running:

```bash
ls /sbin/mount_* | cut -c13-
```

And that's all. Hope it saves time for you, and it will save time for me from now on :laughing:.
