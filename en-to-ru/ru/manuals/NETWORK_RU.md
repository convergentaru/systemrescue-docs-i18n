# Network configuration and programs

## Network configuration tools

Если у вашей системы supported hardware, Ethernet или Wifi network адаптер должен быть автоматически detected, driver загружен и auto configuration via DHCP attempted.

SystemRescue использует NetworkManager как дефолтный network configuration tool. Он предоставляет очень user friendly graphical interface для конфигурации сети. Он делает конфигурацию легче особенно для wireless сетей. Например, wireless сети будут автоматически detected и connecting к ним очень easy. Within X11 NetworkManager доступен как small icon в taskbar просто next to clock. Он также предоставляет `nmcli` и `nmtui` если хотите конфигурировать сеть используя command line или semi-graphical interface из терминала.

Вы также можете конфигурировать сеть с другими tools такими как стандартные `ifconfig` или `ip` commands. В этом случае вы должны остановить NetworkManager service first, иначе он будет conflict и вы потеряете свои settings. Вы можете остановить его запустив `systemctl stop NetworkManager` в shell.

## Manually configure the network

По умолчанию все network interfaces активированы и автоматически конфигурируются. Команда `nmcli` покажет overview того что currently configured.

Чтобы disable и re-enable interfaces используйте `nmcli conn down "Wired connection 1"` и `nmcli conn up "Wired connection 1"`. Обратите внимание что "Wired connection 1" использован как example здесь не является именем interface как assigned kernel, а name internal NetworkManager. Используйте `nmcli` чтобы получить overview всех connections и их names.

Чтобы вручную set static IP, netmask, gateway вы также можете использовать nmcli:

```
nmcli conn mod "Wired connection 1" ipv4.method manual
nmcli conn mod "Wired connection 1" ipv4.addr 192.168.1.20/24
nmcli conn mod "Wired connection 1" ipv4.gateway 192.168.1.1
nmcli conn mod "Wired connection 1" ipv4.dns 192.168.1.1
nmcli conn up "Wired connection 1"
```

Последний "conn up" required чтобы reconfigure interface с новыми parameters.

Чтобы переключиться back to automatic configuration via DHCP используйте:

```
nmcli conn mod "Wired connection 1" ipv4.method auto
nmcli conn up "Wired connection 1"
```

## Firewall

С версии 6.0.4 SystemRescue приходит с iptables firewall enabled чтобы block incoming connections requests по умолчанию. Вам нужно обновить iptables configuration или остановить iptables и ip6tables services если хотите быть able receive incoming connections из outside. Вы можете boot SystemRescue используя опцию `nofirewall` на command line если не хотите firewall.

## Running an SSH Server

SSH позволяет использовать shell на another computer и вы можете copy files (с scp или rsync over ssh). SSHd уже running в background. Но чтобы allow incoming connections, вы должны allow connection через firewall (см. above) и изменить root password. Просто наберите `passwd` чтобы set password. Вы также можете использовать boot options `rootpass=xxx` или `rootcryptpass=xxxx` перед SystemRescue starts чтобы define root password.

Более secure alternative — использовать SSH key files. Вы можете install authorized keys на ваш SystemRescue media с [YAML file](/manual/Configuring_SystemRescue_sysconfig/).

## Accessing a Share on a Windows computer with CIFS

SystemRescue приходит с smbfs/cifs client package который позволяет connect к Windows machine having shared drives. В recent kernels, support для smbfs был replaced с cifs так что вы должны try не использовать smbfs.

Package mount-cifs позволяет access Windows computer на network. Вот example чтобы объяснить как access Windows shared folders. Consider Windows box на 192.168.10.3 и имеет shared directory названный mydata accessible пользователем robert:

```
mkdir /mnt/windows
mount -t cifs //192.168.10.3/mydata /mnt/windows -o username=robert,password=passwd
cd /mnt/windows
```

Теперь вы должны быть able видеть files в /mnt/windows. Не забудьте umount directory когда закончите что делаете в shared directory.

```
umount /mnt/windows
```

## Mounting remote SSH shares as local file systems

Если хотите access files located на remote system к которому можно connect via SSH, вы можете mount remote directory с ssfs и FuSE library. "Userland FileSystem" позволяет mount share, работать на remote files как вы работаете на any local files. С этими file systems, вы можете umount share с standard `umount` command. Вот example как mount SSH file system в /mnt/ssh как anonymous (read only):

```
mkdir /mnt/ssh
passwd root
sshfs [email protected]:/path/to/dir /mnt/ssh
cd /mnt/ssh
umount /mnt/ssh
```

## Accessing cloud storage (WebDAV, S3 and others)

SystemRescue содержит program [rclone](https://rclone.org) который позволяет access, sync и mount многие different cloud storage services. Чтобы использовать его вы должны configure storage service с `rclone config` в interactive menu first. Каждый service assigned remote name таким образом.

После этого используйте `rclone ls <remote name>:` чтобы list remote contents.

Используйте `rclone copy /local/path  <remote name>:destpath` чтобы upload файл и `rclone copy <remote name>:sourcefile /local/targetpath` чтобы download.

---

**Дата перевода:** 2026-11-03  
**Статус:** Переведено (1 документ)  
**Источник:** https://www.system-rescue.org/manual/Network_configuration_and_programs/
