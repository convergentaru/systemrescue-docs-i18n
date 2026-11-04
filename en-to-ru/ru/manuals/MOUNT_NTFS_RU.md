# Mounting NTFS file systems

## Introduction

NTFS (New Technology File System) является дефолтным файловая система используемая на recent Windows версиях и Windows system disk основан на NTFS format.

Linux предоставил full read support для NTFS filesystems уже long time. Однако, до recently, NTFS write support который came с kernel был missing. Чтобы получить доступ к full read-write support для NTFS disks, вы должны были использовать [ntfs-3g](https://wiki.archlinux.org/title/NTFS-3G), который является userspace program, и hence он очень slow.

SystemRescue версия 9.00 приходит с Linux-5.15 который приносит [kernel based ntfs3 file system driver](https://www.kernel.org/doc/html/latest/filesystems/ntfs3.html). Этот modern kernel module предоставляет full read-write support для NTFS на Linux. Это recommended способ чтобы access NTFS disks из SystemRescue. На этом этапе ntfs-3g всё ещё является дефолтным NTFS driver, так что вы должны явно указать что хотите использовать ntfs3 чтобы benefit от этого new driver.

Если вы не знаете имя device который содержит NTFS file system, вы должны использовать `lsblk` и `blkid` или GParted чтобы identify disk. В следующих examples мы'll assume device name `/dev/xxx`, пожалуйста replace это с именем actual device.

## Full Read-Write support

Если вам нужен complete NTFS Write support, должен использовать mount command типа:

```
mkdir -p /mnt/windows
mount -t ntfs3 /dev/xxx /mnt/windows
```

## Read only support

Если вы просто хотите read support на NTFS disks, вы можете указать опцию `-o ro` чтобы restrict access к filesystem). Этот read-only support safer (вы не можете alter или damage ваши данные) и может быть использован например если вы просто хотите backup несколько файлов, или read документ.

```
mkdir -p /mnt/windows
mount -t ntfs3 -o ro /dev/xxx /mnt/windows
```

---

**Дата перевода:** 2026-11-03  
**Статус:** Переведено (1 документ)  
**Источник:** https://www.system-rescue.org/manual/Mounting_ntfs_filesystems/
