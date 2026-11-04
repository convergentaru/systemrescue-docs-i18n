# PXE network booting

## About

[Покружение PXE](https://en.wikipedia.org/wiki/Preboot_Execution_Environment) позволяет загружать компьютер с SystemRescue даже если у этого компьютера нет CDRom drive или USB socket. Это позволяет загружать SystemRescue из сети, и затем быстрее troubleshooting компьютеров на вашей сети раз один PXE-boot-server установлен. Это также полезно если вы хотите загрузить компьютер который не имеет оптический привод (CD или DVD) или troubleshooting удалённый компьютер к которому у вас нет физического доступа вставить диск.

Инструкции ниже соответствуют файлам с 64 bit версией (amd64/x86_64) SystemRescue. Если вы используете 32 bit версию вам нужно заменить все инстансы `x86_64` на `i686`.

## Requirements

* **Servers to provide PXE services** включая DHCP с SystemRescue файлами доступными для serve клиент requests
* **A client computer** с PXE able network card на той же сети и достаточно памяти полностью хранить SystemRescue файлы

## How the PXE boot process works

### The PXE boot server

PXE boot сервер состоит из трёх этапов:

* **stage0:** PXE-booting клиент отправляет extended DHCPDISCOVER
* **stage1:** DHCP сервер отправляет IP адрес клиенту и дополняющую информацию такую как TFTP сервер
* **stage2:** PXE-booting клиент конфигурирует сеть и запрашивает первые boot файлы с TFTP сервера (boot loader, kernel image и sysresccd.img)
* **stage3:** PXE-booted клиент запрашивает squashfs root файловую систему образ с HTTP или NFS или NBD сервера

Эти три части могут быть установлены либо на single machine или на нескольких компьютерах.

### The PXE boot process

Вы можете понять что происходит когда загружаете SystemRescue из сети. Вам потребуется это знание для troubleshooting в случае проблем. Вот самые важные шаги PXE boot процесса:

* Когда клиентский компьютер пытается загрузиться с PXE, он сначала emits DHCP request на сети чтобы получить IP адрес.
* Затем DHCP сервер отвечает с DHCP offer который содержит новый IP адрес который ещё не был allocated и некоторые специфические опции (DNS, default route) и IP адрес TFTP сервера.
* Клиент получает этот DHCP offer и принимает его. Он подключается к TFTP серверу (он получил свой IP адрес в предыдущем этапе) чтобы получить boot loader файлы.
* TFTP сервер отправляет boot loader файлы (pxelinux<!-- raw HTML omitted -->) и текстовые файлы отображаемые на экране pxelinux<!-- raw HTML omitted -->.</li>
* Клиент отображает pxelinux prompt, и пользователь может выбрать boot опции. Затем он запрашивает с TFTP сервера kernel и initramfs файлы необходимые чтобы загрузить систему.
* TFTP сервер отправляет kernel и initramfs файлы (например vmlinuz<!-- raw HTML omitted --> + sysresccd.img<!-- raw HTML omitted -->) клиенту.
* Клиент загружает этот kernel и выполняет init программы которые приходят с initramfs.
* Во время его initialization kernel делает DHCP request снова из-за ip=dhcp<!-- raw HTML omitted --> kernel boot parameters. Действительно kernel не знает IP адрес используемый компьютером на pxelinux этапе.
* Клиенту нужен файл airootfs.sfs<!-- raw HTML omitted -->. Если вы используете HTTP или TFTP для третьего этапа, то airootfs.sfs<!-- raw HTML omitted --> будет downloaded into RAM так что клиент должен иметь достаточно памяти (estimated requirement: 1GB). Если вы используете либо NFS или NBD тогда у вас нет этого memory requirement и клиент будет делать permanent requests к серверу каждый раз когда ему нужен файл с root filesystem.
* Клиент монтирует airootfs.sfs<!-- raw HTML omitted --> и он теперь может завершить boot процесс.
* На этом этапе клиент держит все файлы в памяти, если вы использовали TFTP/HTTP для третьего этапа, так что он не требует boot сервера больше. Если вы используете NFS или NBD, соединение всё ещё требуется.

### Customization of the boot command line

PXE сервер состоит из нескольких сервисов. На втором этапе, сервер использует TFTP чтобы отправить multiple вещи клиенту: boot loader (pxelinux.0), kernel image (vmlinuz), initramfs (sysresccd.img). Boot loader является pxelinux.0<!-- raw HTML omitted --> и он приходит с configuration file который отправляется клиенту. Этот configuration file содержит boot command line которая будет использоваться клиентом чтобы запустить Linux kernel. Эта command line важна поскольку она содержит SystemRescue boot опции которые требуются для запуска третьего этапа. Boot command line может быть использована чтобы указать network settings и метод который PXE клиент будет использовать на третьем этапе PXE boot процесса. Вот два примера валидных command lines для PXE boot:

### PXE boot options supported by SystemRescue

SystemRescue основан на Arch Linux с версии 6.0 hence он поддерживает PXE boot опции реализованные upstream. Эти boot опции предоставляют поддержку различных протоколов (HTTP, NFS, NBD) и документированы на следующей странице: [https://gitlab.archlinux.org/archlinux/archiso/-/raw/v43/docs/README.bootparams](https://gitlab.archlinux.org/archlinux/archiso/-/raw/v43/docs/README.bootparams)

Самые важные из них следующие:

#### HTTP option

* **archiso_http_srv=** Set a HTTP URL (должен заканчиваться /) где download *.sfs файл когда использовать HTTP. ${archisobasedir} и архитектура (`x86_64`) добавляются к supplied URL. http и https поддерживаются. Когда используете https, certificate check не делается как initramfs environment не содержит trusted CA database.

#### NFS option

* **archiso_nfs_srv=** Set the NFS-IP:/path сервера где download *.sfs файл когда использовать NFS. ${archisobasedir} и архитектура (`x86_64`) добавляются к supplied path. Обратите внимание что из-за tooling limitations только IPv4 адреса поддерживаются, нет DNS hostnames или IPv6.

### Example of a PXE configuration to boot SystemRescue using TFTP and HTTP

Вот пример PXE конфигурации которая позволяет загрузить SystemRescue используя только TFTP и HTTP. Это одна из самых простых PXE конфигураций которую вы можете иметь hence рекомендуется следовать этому примеру если хотите простую PXE конфигурацию для SystemRescue.

В этом примере boot loader является pxelinux.0<!-- raw HTML omitted --> и TFTP используется чтобы получить kernel image и initramfs images. Затем использует HTTP сервер чтобы download большой squashfs файл sysresccd airootfs.sfs<!-- raw HTML omitted -->.

Когда конфигурируете TFTP сервер вам нужно скопировать boot loader (pxelinux binary файлы), pxelinux configuration file в `pxelinux.cfg/default`, и вам нужно скопировать директорию `sysresccd` как есть (хотя не все файлы требуются TFTP серверу) с ISO image чтобы предоставить файлы которые будут запрашиваться via TFTP.

Вот список всех required файлов и их location на TFTP сервере:

```
./sysresccd/boot/x86_64/vmlinuz
./sysresccd/boot/x86_64/sysresccd.img
./sysresccd/boot/memtest.COPYING
./sysresccd/boot/memtest
./sysresccd/boot/intel_ucode.LICENSE
./sysresccd/boot/intel_ucode.img
./sysresccd/boot/amd_ucode.LICENSE
./sysresccd/boot/amd_ucode.img
./sysresccd/VERSION
./sysresccd/pkglist.x86_64.txt
./pxelinux.cfg/default
./ldlinux.c32
./pxelinux.0
```

Вот пример configuration расположенного в `pxelinux.cfg/default`:

```
DEFAULT sysresccd
LABEL sysresccd
  LINUX sysresccd/boot/x86_64/vmlinuz
  INITRD sysresccd/boot/intel_ucode.img,sysresccd/boot/amd_ucode.img,sysresccd/boot/x86_64/sysresccd.img
  APPEND archisobasedir=sysresccd ip=dhcp archiso_http_srv=http://10.0.2.4/ checksum
  SYSAPPEND 3
```

Опция SYSAPPEND важна и вам нужно убедиться что вы используете recent version pxelinux как версии older than 5.10 не поддерживают её. Также пользователи PXE boot feature encouraged использовать recent versions SystemRescue.

Вам нужно обновить IP адрес HTTP сервера с которого squashfs image будет downloaded. В этом примере он будет download этот файл и его checksum из следующих URLs:

```
http://10.0.2.4/sysresccd/x86_64/airootfs.sfs
http://10.0.2.4/sysresccd/x86_64/airootfs.sha512
```

Убедитесь что вы можете успешно download эти файлы с regular web client до того как пытаться boot из PXE. Вы можете customize boot command line добавляя опции такие как `setkmap=us` после опции `checksum`.

---

**Дата перевода:** 2026-11-03  
**Статус:** Переведено (1 документ)  
**Источник:** https://www.system-rescue.org/manual/PXE_network_booting/
