# Установка SystemRescue на диск

## Обзор

Эта страница объясняет как установить SystemRescue на локальный диск чтобы он мог быть запущен без необходимости использовать съёмное устройство. Эти инструкции валидны для SystemRescue с версии 6.0.0.

## Introduction

Существует лёгкий способ запустить SystemRescue с локального диска. Это может быть удобно если вы часто хотите boot на SystemRescue так как это будет быстрее и нет необходимости insert любое media перед перезапуском вашего компьютера. Этот метод позволяет использовать SystemRescue как вы делаете с CDRom или USB версией. Он также позволяет запустить SystemRescue если у вас нет CD/DVD drive или USB socket в вашем компьютере. Вы можете использовать другой operating system для обработки установки SystemRescue, и enable его. Так, этот метод рекомендуется если вы часто используете SystemRescue и хотите избегать необходимости insert устройство каждый раз.

Установка делается копированием файлов с SystemRescue media на локальную файловую систему где установлен ваш operating system. Этот подход не требует любого repartitioning вашего диска. Используя grub4dos вы даже можете установить SystemRescue на Windows NTFS partition. Всё что вам нужно сделать — это установить файлы которые на SystemRescue disc на existing partition на вашем локальном диске, который может быть либо Linux partition (ext4, xfs, …) или Windows один (NTFS). Таким образом вы можете запустить SystemRescue с вашего Windows partition, и используя опцию `copytoram` вы даже можете использовать его чтобы troubleshoot Windows сам или `ntfs-3g` чтобы работать на файловую систему где он установлен.

Есть два шага в этой установке: сначала вам нужно скопировать main SystemRescue files onto partition вашего диска. Затем, вы должны configure ваш boot loader. Процесс установки требует partition с достаточно space чтобы скопировать SystemRescue файлы. Вы должны иметь working Grub2 boot loader если хотите установить SystemRescue файлы на Linux partition.

## Первый подход используя Grub2 с loopback.cfg

Grub2 предоставляет feature чтобы boot из ISO image который stored на локальном диске. Если вы положите copy `systemrescue-x.y.z.iso` на файловую систему которую Grub2 может read то вы можете запустить SystemRescue напрямую из ISO image stored на вашем диске. Это очень удобно если вы frequently обновляете SystemRescue и хотите boot его напрямую из Grub2 или когда хотите setup SystemRescue на working system как backup boot option. Так даже если normal operating system становится unusable, пока GRUB и appropriate hard drive partition в working order, вы'll be able запустить SystemRescue.

Для этого SystemRescue реализует convention `loopback.cfg` как [documented here](https://supergrubdisk.org/wiki/Loopback.cfg).

### grml-rescueboot

Самый лёгкий способ использовать это — через package [grml-rescueboot](https://aur.archlinux.org/packages/grml-rescueboot) который доступен для Arch via AUR и Debian/Ubuntu. Установите его, положите SystemRescue ISO-image file в `/boot/grml/`, и запустите `update-grub`. Next time вы enter системы GRUB menu, вы'll see новый item. Select it, и submenu со standard SystemRescue boot options появится. Pressing Esc вернёт вас назад к previous menu.

Если у вас нет достаточно disk space в `/boot/` чтобы accommodate ISO image, возможно потому что он на small partition, то **grml-rescueboot** может быть configured чтобы look для ISO images в different location. Смотрите параметр `ISO_LOCATION` в файле `/etc/default/grml-rescueboot`.

### Custom config

Альтернативно, если вы comfortable с GRUB configuration language, boot stanza как следующий должен сделать the trick. Убедитесь что edit line `iso_path=` чтобы reflect имя и location вашего SystemRescue ISO-image file:

```
menuentry "SystemRescue" {
  iso_path="/boot/iso/systemrescue.iso"
  export iso_path
  search --set=root --file $iso_path
  # support booting recent GRUB versions on UEFI systems
  rmmod tpm
  loopback loop $iso_path
  set root=(loop)
  # the path below is on the SystemRescue ISO, not the hard drive
  configfile /boot/grub/loopback.cfg
}
```

Это обычно может быть добавлено в `/boot/grub/custom.cfg` или `/etc/grub.d/40_custom`.

Обратите внимание что:

1. Рекомендуется удалить version number form имени ISO file чтобы вы не должны update Grub configuration каждый раз когда download новый version ISO image.
2. Если вы edit `40_custom`, то вы'll usually need запустить команду `update-grub`, `grub-mkconfig -o /boot/grub/grub.cfg` или `grub2-mkconfig -o /boot/grub2/grub.cfg` (зависимо от used Linux distribution) после этого
3. Вы может быть нужды загрузить некоторые GRUB modules (например `lvm`) чтобы GRUB был able read disk location containing ISO-image file
4. ISO path relative к его containing filesystem, и может не match путь который вы see на running system. Например если `/boot/` на separate partition, то правильный путь выше будет `/iso/systemrescue.iso`, без `/boot` prefix.) More information на installing и configuring GRUB может быть found [here](https://wiki.archlinux.org/title/GRUB).

### Steps during boot

Важно понять main steps boot процесса чтобы получить configuration right:

* Grub2 searches через все accessible partitions для filesystem который содержит файл с именем `/boot/iso/systemrescue.iso`
* Путь к этому filesystem затем stored в переменную `root`
* ISO file opened via loopback mechanism
* Переменная `root` теперь set использовать loopback device
* Config file `/boot/grub/loopback.cfg` который contained внутри SystemRescue загружен и выполнен
* Этот config file будет использовать переменную `iso_path` чтобы определить правильный `archiso_param` variable
* Затем regular `/boot/grub/grubsrcd.cfg` grub config из SystemRescue загружен и выполнен

## Alternative approach involving extracting contents from the ISO image

Этот подход работает на Linux с Grub boot loader и он также работает на Windows с Grub4dos.

Он involves копирование SystemRescue main files на filesystem на disk и configuring Grub или Grub4dos чтобы он мог быть booted.

### Первый шаг: copy files from the ISO image to the disk

Теперь, mount ISO image с `mount` under Linux (например `mount -o loop systemrescue-x.y.z.iso /mnt/cdrom`) или вы можете использовать software типа Daemon-Tools под Windows если не burned disc), в order получить main файлы.

Вы должны скопировать всю директорию `/sysresccd/` из ISO на root folder partition где это installed. Если у вас separate `/boot` partition тогда эта directory должна быть скопирована в `/boot/sysresccd`. Вы должны сохранить directory structure как она на original ISO.

### Второй шаг: update the boot loader configuration (если вы install on Linux)

Теперь, вы должны update boot loader. Эта section описывает как update grub. Если вы используете NTFS partition, пожалуйста read следующую section вместо этого.

Мы'll have to добавить несколько lines к configuration file boot manager (обычно located в `/etc/grub.d/` для Grub2). Вы должны customize configuration given там.

Вот пример Grub2 configuration. Создайте новый файл типа `/etc/grub.d/25_sysresccd` чтобы его configuration был located после вашего default operating system configuration. Search directive очень important так как он provides способ grub locate filesystem который содержит SystemRescue файлы.

В следующем примере filesystem идентифицирован используя его label который `boot` но вы также можете identify filesystem используя его UUID если предпочитаете. Once Grub finds device с этим label он will set it как его root filesystem. Это может быть confusing как что grub considers как root filesystem это то что Linux будет consider как boot filesystem если `/boot` на separate filesystem. Paths к SystemRescue files relative к root этого filesystem который может быть different от Linux path.

Опция `archisolabel=boot` указывает что SystemRescue попытается найти свои файлы на filesystem который labelled `boot` как grub. Опция `archisobasedir=sysresccd` затем указывает что SystemRescue needs искать свои файлы в `/sysresccd` на этом filesystem. Опция `copytoram` рекомендуется чтобы boot filesystem мог быть unmounted после boot процесс завершен, который позволяет вам perform changes на disk где SystemRescue установлен.

```
#!/bin/sh
exec tail -n +3 $0

menuentry 'SystemRescue' {
  load_video
  insmod gzio
  insmod part_gpt
  insmod part_msdos
  insmod ext2
  search --no-floppy --label boot --set=root
  echo   'Loading Linux kernel ...'
  linux  /sysresccd/boot/x86_64/vmlinuz archisobasedir=sysresccd archisolabel=boot copytoram setkmap=us
  echo   'Loading initramfs ...'
  initrd /sysresccd/boot/x86_64/sysresccd.img
}
```

После создания configuration file обычно нужно запустить команду типа `grub-mkconfig -o /boot/grub/grub.cfg` чтобы produce final grub configuration. Убедитесь что final grub configuration содержит expected section для SystemRescue перед reboot.

### Второй шаг: update the grub4dos bootmanager (если вы install on Windows)

Теперь, вы должны update ваш bootmanager используя grub4dos который является grub port to windows. Эта section описывает как установить grub4dos boot manager если вы installed SystemRescue файлы на NTFS partition running Windows. Если вы используете Linux partition, пожалуйста read предыдущую section вместо этого.

Один из most interesting вещей что вы можете сделать с sysresccd ntfs installation — troubleshoot windows когда он has проблемы. Таким образом вы можете mount windows partition с ntfs-3g и repair ваш windows (replace backup registry, …). Единственная проблема вы не можете mount windows disk read-write с ntfs-3g потому что он был уже mounted read-only во время boot процесс. Решение этой проблемы — использовать опцию `copytoram` на boot time. Когда эта option enabled, sysresccd будет cache свои файлы (found на ntfs disk) в memory во время boot процесс, и ntfs disk будет unmounted. Так он позволяет вам mount его снова с ntfs-3g. Так вы должны добавить `copytoram` к `menu.lst` boot options если хотите быть able mount ваш windows disk с ntfs-3g после booting из ntfs disk сам.

Installation really straight forward. В этом mini tutorial, я assume Windows установлен на NTFS disk (Disk-C) и что вы copied SystemRescue main files into `C:\sysresccd`

Вы должны download grub4dos, extract zip file в temporary directory, и скопировать `grldr` к `C:\`. Эта installation была tested используя файл `grldr` provided с `grub4dos-0.4.3-2007-08-27.zip` но он should work с any recent version.

Вместе с `grldr` вы need скопировать `grldr.mbr` (part archive) на root Windows boot partition. Затем вы need набрать несколько commands в command prompt (run `cmd.exe` с administrator user account).

In command prompt window (`C:>` dummy substitute cmd prompt) do following:

```
C:> bcdedit /create /d "SystemRescue [GRUB4DOS]" /application bootsector
```

Вы get return boot entry `{id}` — use it (copy/paste или type) в следующих steps:

```
C:> bcdedit /set {id} device boot
C:> bcdedit /set {id} path \grldr.mbr
C:> bcdedit /displayorder {id} /addlast
```

Вы должны создать `C:\menu.lst` который является `grub4dos` configuration file. Вот пример. Он соответствует 64 bit architecture (amd64/x86_64). Если вы используете 32 bit version вы need заменить все инстансы `x86_64` с `i686`:

```
# This is a sample menu.lst file for SystemRescue
title    SystemRescue from the NTFS disk
root     (hd0,0)
kernel   /sysresccd/boot/x86_64/vmlinuz archisobasedir=sysresccd archisolabel=windows copytoram setkmap=us
initrd   /sysresccd/boot/x86_64/sysresccd.img
```

In `menu.lst` вы'll have to update `archisolabel` чтобы it matches label NTFS filesystem который содержит `sysresccd` и также вы should update параметр `setkmap` чтобы code matches вашей keyboard configuration. В этом примере, files located на NTFS partition которая является first partition первого hard disk. Затем grub device name `(hd0,0)`. Оно будет `(hd1,0)` для first partition второго hard-disk, `(hd0,1)` для second partition первого hard-disk, … Вы должны replace `root (hd0,0)` с grub имени вашего NTFS partition.

---

**Дата перевода:** 2026-11-03  
**Статус:** Переведено (1 документ)  
**Источник:** https://www.system-rescue.org/manual/Installing_SystemRescue_on_the_disk/
