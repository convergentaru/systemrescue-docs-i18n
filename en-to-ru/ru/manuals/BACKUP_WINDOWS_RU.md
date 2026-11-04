# Резервное копирование данных с неработающего Windows компьютера

## Об этом туториале

Этот туториал объясняет как вы можете сделать backup данных от Windows компьютера который больше не может запускаться. Есть много причин почему Windows компьютер мог перестать работать: установка плохого драйвера, системный крах с disk corruption, файлы системы удалены, ... В таком случае вы можете просто переустановить операционную систему, но самое важное — сначала сделать backup ваших данных. Если Windows не загрузочна, вы можете просто загрузиться используя SystemRescue для этого. Даже running Linux, она способна читать Windows диск, и затем вы сможете backup ваши данные через network. Все шаги подробно описаны в этом туториале так что Windows пользователи без знаний Linux могут следовать ему для backup их данных.

Если это первый раз когда вы используете SystemRescue, рекомендуется чтобы вы прочитали [Quick start Guide](/Quick_start_guide/) для избежания проблем.

## Обзор

SystemRescue — это livecd. Она содержит полную операционную систему based на Linux. Это означает что вы сможете манипулировать файлами от её окружения даже если Windows OS, которая установлена на этом компьютере, сломана. Конечно, вам понадобится working компьютер для download и burn SystemRescue.

* **Шаг-1: prepare загрузочный cdrom/usb-key**
  Сначала вы должны скачать и записать (burn) SystemRescue. Для этого нужен working компьютер. Если вы не можете запустить SystemRescue от cdrom, вы также можете [Install SystemRescue на USB stick](/Installing-SystemRescue-on-a-USB-memory-stick/) и запуститься от него.
* **Шаг-2: get доступ к вашему Windows диску**
  В этом разделе мы увидим как читать Windows disk(s) от Linux, чтобы увидеть какие файлы могут быть backed up.
* **Шаг-3: backup ваши данные через network**
  Если у вас есть network access (Ethernet card и другой компьютер на который данные могут быть скопированы) этот раздел объясняет как конфигурировать network и copy данные к другому box.
* **Шаг-4: reboot правильно**

## Шаг-1: prepare загрузочный cdrom или usb-key

Всё что вы должны сделать — это скачать latest ISO image SystemRescue для x86, записать (burn) его onto cdrom disc используя любой burning софт. Если компьютер где Windows сломана не может запуститься от cdrom disc, вы также можете [Install SystemRescue на USB stick](/Installing-SystemRescue-on-a-USB-memory-stick/) и запуститься от того.

Затем, войдите в BIOS вашего компьютера (вам часто нужно press F2 или DEL во время hardware initialisation периода), войдите в section где boot order определён, и убедитесь что cdrom или USB device — первый device в списке (или перед hard drive).

Теперь, reboot с cdrom / usb stick inserted. Если это работает, вы должны увидеть SystemRescue ASCII art logo, и boot инструкции. Вы затем можете просто press enter для запуска SystemRescue с default settings. Если вы планируете использовать network, рекомендуется запуститься с вашими network settings так что вам не нужно конфигурировать его позже. Для этого, просто введите следующую строку на prompt и press enter. Конечно, вы должны заменить этот address с relevant IP address для этого компьютера:

```
rescue64 ethx=192.168.1.158 gateway=192.168.1.254
```

Gateway address требуется только если вы хотите подключиться к компьютеру который расположен на different network. Для использования dynamic IP address вместо static, вы можете ввести следующее:

```
rescue64 dodhcp
```

Не волнуйтесь о network settings сейчас, вы можете изменить их позже если это fails. Вы также можете прочесть главу о [boot options](/manual/Booting_SystemRescue/) для получения больше деталей об этом.

## Шаг-2: get доступ к вашему Windows диску

Когда boot процесс завершён, вы должны получить black screen на которой type команды. Все следующие шаги будут based on commands так что вы можете остаться в этом окружении. Вы также можете run команды от terminal в graphical окружении если вы предпочитаете. Просто введите <code>startx</code> в таком случае.

### Обнаружение дисков (Detection of the disks)

Первое что нужно сделать — это обнаружить partition где ваши данные расположены. Linux использует names которые different от Windows для дисков, так что вы увидите names такому как <code>/dev/sda1</code>, <code>/dev/sda2</code> вместо <code>C:</code> или <code>D:</code>. Для обнаружения дисков и partitions, просто run следующую команду: <code>fsarchiver probe simple</code>. Вы должны увидеть output такого:

```
root@sysresccd /root % fsarchiver probe -v
[=====DEVICE=====] [==FILESYS==] [=====LABEL=====] [====SIZE====] [MAJ] [MIN]
[/dev/sda1       ] [ntfs       ] [Windows-XP     ] [    25.00 GB] [  8] [  1]
[/dev/sda5       ] [ntfs       ] [Data           ] [   120.00 GB] [  8] [  5]
[/dev/sda6       ] [ntfs       ] [Backup         ] [    70.00 GB] [  8] [  6]
```

В этом примере было три partitions на компьютере, у вас может быть только одна или две partitions. Первый столбец очень important поскольку это device name partition которую мы будем need для доступа к вашим данным. Вы должны записать это name на piece of paper.

### Доступ к correct partition

Теперь мы будем have to сделать эти данные доступными от системы. Давайте consider что вы хотите backup данные расположенные на диске called <code>/dev/sda1</code>. Первое что нужно сделать — это mount partition к directory. Это просто означает что эта directory будет contain view вашего диска, ничего не будет written к нему:

```
ntfs-3g -o ro /dev/sda1 /mnt/windows
```

Единственная опция которая используется здесь — <code>ro</code>, она означает что данные на диске mounted с read-only access. Мы просто используем эту опцию для того чтобы убедиться мы не можем сделать любые ошибки которые могут destroy эти данные, но вы можете mount диск с full read-write access если вы предпочитаете (просто remove эту опцию от команды). Теперь, давайте посмотрим contents partition:

```
cd /mnt/windows
ls -l
```

Вы должны увидеть что-то такое:

```
root@sysresccd /mnt/windows % ls -l
total 132856
-r-------- 1 root root      245 2008-08-05 19:36 boot.ini
dr-x------ 1 root root     4096 2008-07-02 07:34 cygwin
dr-x------ 1 root root     4096 2007-10-23 21:18 Documents and Settings
-r-------- 1 root root    47772 2005-03-25 12:00 NTDETECT.COM
-r-------- 1 root root   297072 2007-10-21 18:50 ntldr
-r-------- 1 root root 16777216 2008-09-22 16:41 pagefile.sys
dr-x------ 1 root root     4096 2008-08-10 19:07 Program Files
dr-x------ 1 root root        0 2008-07-02 07:45 RECYCLER
dr-x------ 1 root root     4096 2007-10-21 17:59 System Volume Information
dr-x------ 1 root root    32768 2008-08-29 22:48 WINDOWS
```

Если вы хотите browse ваши данные, рекомендуется использовать Midnight Commander. Просто введите <code>mc</code> в console для получения его. Это очень intuitive tool similar к Norton Commander который может быть использован для browse вашего диска. Он также может быть использован для copy, move, rename файлы и directories используя function keys. Press F10 для выхода от Midnight Commander.

## Шаг-3: backup ваши данные через network

Есть multiple ways для backup ваших данных. Using network recommended поскольку это quick и оно limited только amount of space который есть на другом компьютере. Basic network configuration не complex под Linux если у вас есть ethernet card.

### Конфигурация IP network

Сначала, давайте проверим какие cards обнаружены. Нет драйвера для install. Linux comes со всеми network drivers по default. Ethernet cards называются <code>eth0</code>, <code>eth1</code>, <code>eth2</code>, ... под Linux. Если у вас только одна card она будет <code>eth0</code>. Если у вас больше что одна card, помните что первая может не быть той которую вы expect. В таком случае вы можете просто конфигурировать все cards с IP address и это должно work. Введите следующую команду:

```
ifconfig -a
```

Вы должны увидеть screen такого:

```
root@sysresccd /mnt/windows % ifconfig -a
eth0      Link encap:Ethernet  HWaddr 00:18:f3:ce:0e:36
          inet addr:192.168.1.158  Bcast:192.168.1.255  Mask:255.255.255.0
          UP BROADCAST MULTICAST  MTU:1500  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:0 (0.0 B)  TX bytes:0 (0.0 B)
          Interrupt:16 Base address:0xe000
```

Если IP address ещё не configured, вы можете set static IP address (и gateway если required) следующим:

```
ifconfig eth0 192.168.1.158 netmask 255.255.255.0
route add default gw 192.168.1.254
```

Для получения dynamic address через DHCP, просто дайте команду:

```
dhclient eth0
```

### Конфигурация SFTP сервера

В этом разделе мы будем consider что сломанный компьютер использует 192.168.1.158 и компьютер на который вы хотите copy данные — 192.168.1.20.

Multiple protocols могут быть использованы для transfer ваших данных. В этом туториале мы будем использовать SCP/SSH protocol по нескольким причинам:

* оно использует только single TCP connection на port 22 который легче конфигурировать чем FTP который требует two connections для работы
* оно reliable, и оно encrypted так что нет risk к вашим данным если вы не доверяете people подключённые к вашей local network.
* оно очень easy to configure на client и server side (samba может быть использован для share ваших файлов но конфигурация может быть tricky)

Сломанный компьютер run OpenSSH сервер который означает другой компьютер может подключиться к нему используя tcp/22 port طالما он knows пароль. Компьютер на который данные будут copied будет have to run SFTP client. Мы recommend Filezilla-client который является very good free и multiplatform FTP/SFTP client. Вы можете использовать его exactly как любой другой FTP client за исключением protocol будет SFTP. Вы также могли бы использовать WinSCP если вы предпочитаете.

Теперь, мы будем have to set пароль на server side (на linux) так что мы можем подключиться от другого end. Просто введите следующую команду, и введите пароль twice (он не будет displayed на screen по security reasons):

```
passwd
```

### Конфигурация SFTP клиента

Теперь сервер готов, вы можете пойти к другому компьютеру (который работает), install Filezilla-client к нему (Filezilla works на Windows и Linux), и сделайте это:

* кликните на File/Site-manager в menu bar
* кликните на new-site для создания новой connection
* заполните dialogue window следующей информацией:
  * Host: 192.168.1.158 (IP address компьютера running SystemRescue)
  * Port: 22
  * Server type: SFTP (SSH File Transfer Protocol)
  * Longon type: Normal
  * Login: root
  * Password: введите пароль который вы ввели в previous step.
  * Кликните на «Advanced»
  * Default Directory: /mnt/windows
* Теперь кликните на connect

Вы теперь должны быть connected к OpenSSH серверу running на SystemRescue, и вы должны увидеть local disk на одной стороне окна, и remote disk на другой стороне. Если вы не видите ваши remote файлы, вам нужно пойти в directory <code>/mnt</code>, и затем в <code>windows</code>.

## Шаг-4: reboot ваш компьютер

Когда вы закончили использовать SystemRescue, рекомендуется unmount вашу Windows partition, и затем reboot computer правильно:

```
umount /mnt/windows
reboot
```

---

**Дата перевода:** 2026-11-03  
**Статус:** Переведено (1 документ)  
**Источник:** https://www.system-rescue.org/manual/Backup_data_from_an_unbootable_windows_computer/
