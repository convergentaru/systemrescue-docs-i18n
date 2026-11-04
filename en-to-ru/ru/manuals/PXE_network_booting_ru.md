+++
title = "Сетевая загрузка через PXE"
draft = false
---

## Обзор PXE

PXE (Preboot eXecution Environment) — это стандарт для сетевой загрузки компьютеров. SystemRescue поддерживает PXE-загрузку, что позволяет загружать систему через локальную сеть без необходимости наличия загрузочного носителя на каждом компьютере.

## Требования для PXE-сервера

Для настройки PXE-сервера вам понадобится:
* Сервер Linux с установленным DHCP и TFTP
* Сетевое устройство для хранения образа SystemRescue
* Правильная конфигурация DHCP для передачи PXE-запросов

### Необходимые пакеты на сервере

Установите необходимые пакеты:
```bash
pacman -S dnsmasq tftp-hdd wget
```

## Настройка DHCP

### Конфигурация dnsmasq

Создайте или отредактируйте файл конфигурации `/etc/dnsmasq.conf`:

```
# IP-адрес сервера
interface=eth0

# Диапазон адресов для PXE-клиентов
dhcp-range=192.168.1.50,192.168.1.200,12h

# Включение PXE-поддержки
dhcp-option=3,192.168.1.1
dhcp-boot=pxelinux.0

# ТFTP-корень (где искать файлы загрузки)
tftp-root=/var/lib/tftpboot
```

### Настройка опций для SystemRescue

Добавьте специальные опции для SystemRescue:
```
dhcp-option=66,192.168.1.1
dhcp-option=67,/sysresccd/isolinux/sysresccd.cfg
```

## Подготовка файлов загрузки

### Структура директорий TFTP

Создайте структуру директорий для TFTP:
```bash
mkdir -p /var/lib/tftpboot/{isolinux,grub}
```

### Копирование файлов SystemRescue

Скачайте образ SystemRescue и извлеките файлы загрузки:
```bash
wget http://example.com/systemrescue-x.y.z.iso
cp systemrescue-x.y.z.iso /tmp/
mkdir -p /var/lib/tftpboot/isolinux
cd /tmp
split --bytes=1048576 systemrescue-x.y.z.iso iso-
gunzip iso-* | tar -xv

# Или используйте grml-rescueboot для автоматического извлечения
```

### Файл конфигурации PXE

Создайте файл `pxelinux.0` или `sysresccd.cfg`:

Для ISOLINUX:
```bash
DEFAULT sysresccd
LABEL sysresccd
    KERNEL /sysresccd/boot/i386/vmlinuz
    APPEND archisobasedir=sysresccd archisolabel=SystemRescue setkmap=us
    INITRD /sysresccd/boot/i386/sysresccd.img
```

Для GRUB2:
```bash
menuentry 'SystemRescue' {
  load_video
  insmod gzio
  insmod part_msdos
  insmod ext2
  search --no-floppy --fs-uuid --set=root aaaaaaaa-bbbb-cccc-dddd-eeeeeeeeeeee
  linux16 /sysresccd/boot/x86_64/vmlinuz archisobasedir=sysresccd archisolabel=SystemRescue setkmap=us
  initrd16 /sysresccd/boot/x86_64/sysresccd.img
}
```

## Настройка сетевой карты клиента

### BIOS/UEFI настройки

Убедитесь что сетевая карта включена в BOOT-порядок:
1. Введите BIOS во время загрузки (обычно Del, F2 или F10)
2. Перейдите к разделу Boot
3. Переместите Network/PXE выше других опций

### UEFI PXE

Для UEFI систем вам понадобится EFI-версия загрузчика:
```bash
cp /boot/efi/EFI/sysrescue/sysresccd.efi /var/lib/tftpboot/
```

И обновите конфигурацию с соответствующими параметрами для UEFI.

## Тестирование PXE

### Проверка DHCP

Сначала проверьте что DHCP работает:
```bash
# Прослушивание DHCP-запросов
tcpdump -i eth0 port 67 -n
```

Вы должны увидеть запросы от клиентов.

### Проверка TFTP

Тестируйте доступ к файлам через TFTP:
```bash
tftp 192.168.1.1
get pxelinux.0
exit
```

## Автоматизация с помощью скриптов

### Скрипт обновления файлов загрузки

Создайте скрипт для автоматического обновления файлов PXE при обновлении SystemRescue:

```bash
#!/bin/bash
ISO_URL="http://example.com/systemrescue-latest.iso"
TFTP_ROOT="/var/lib/tftpboot"

wget -O /tmp/sysrescue.iso $ISO_URL
mkdir -p /tmp/sysresccd
cd /tmp
split --bytes=1048576 sysrescue.iso iso-
gunzip iso-* | tar -xv

cp boot/*/sysresccd.img $TFTP_ROOT/isolinux/
cp boot/*/vmlinuz $TFTP_ROOT/isolinux/
```

## Безопасность PXE-сервера

### Ограничение доступа

Ограничьте доступ к PXE только для определённых подсетей:
```bash
# В /etc/dnsmasq.conf
dhcp-range=192.168.1.50,192.168.1.200,12h src=192.168.1.0/24
```

### Проверка файлов TFTP

Регулярно проверяйте файлы в директории TFTP:
```bash
find /var/lib/tftpboot -type f -mtime +7 -delete
```

## Мониторинг использования PXE

### Логирование DHCP-запросов

Включите логирование для отслеживания запросов:
```bash
# В /etc/dnsmasq.conf
log-dhcp
```

### Статистика TFTP

Мониторьте количество загрузок через TFTP:
```bash
tcpdump -i eth0 port 69 -nn | grep 'tftp' | wc -l
```

## Заключение

PXE-загрузка — мощный инструмент для массового развертывания SystemRescue. Правильная настройка DHCP, TFTP и файлов загрузки обеспечит надёжную работу сети. Регулярно обновляйте конфигурацию при обновлениях SystemRescue.
