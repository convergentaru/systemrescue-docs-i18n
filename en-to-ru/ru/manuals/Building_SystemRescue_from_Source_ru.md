+++
title = "Сборка SystemRescue из исходного кода"
draft = false
---

## Обзор

SystemRescue можно собрать самостоятельно из исходного кода. Это позволяет:
* Использовать только необходимые компоненты
* Включить собственные пакеты и настройки
* Исправлять ошибки в процессе сборки
* Адаптировать систему под специфические требования

## Требования для сборки

### Минимальные требования

Для сборки SystemRescue вам понадобится:
* Linux-система с доступом к терминалу
* Git для клонирования исходного кода
* Довольно много свободного места на диске (5-10 ГБ)
* Несколько часов времени для компиляции

### Рекомендуемые пакеты

Установите необходимые пакеты:

```bash
# Базовые инструменты сборки
pacman -S git build-base linux-headers linux-any \
  binutils gcc-multilib make patch diffutils \
  xz gzip bzip2 perl python3

# Пакеты для компонентов SystemRescue
pacman -S dosfstools ntfs-3g exfatprogs \
  grub-bios-setup grub-pc-bin syslinux isolinux \
  squashfs-tools xorriso wget curl \
  coreutils grep findutils util-linux \
  xorg-server xterm dbus

# Дополнительные пакеты для расширенных функций
pacman -S gparted partimage fsarchiver rsync \
  networkmanager netctl openvpn \
  php python3-pip sqlite
```

## Клонирование исходного кода

### Из git репозитория

Клонируйте основной репозиторий SystemRescue:
```bash
git clone https://github.com/SystemRescue/SysRescUE.git
cd SysRescUE
```

### Получение последнего релиза

Если хотите собрать конкретную версию:
```bash
git checkout v9.05
```

## Структура исходного кода

Основные директории:
* `sysresccd/` — основная система SystemRescue
* `iso/` — файлы для создания ISO-образа
* `scripts/` — скрипты сборки и установки
* `packages/` — пакеты для системы

## Настройка конфигурации

### Редактирование Makefile

Отредактируйте основной Makefile:
```bash
cd sysresccd
nano Makefile
```

Основные переменные:
```makefile
# Версия SystemRescue
VERSION = "9.05"

# Пакеты для включения (разделите запятыми)
PACKAGES = coreutils,grep,findutils,util-linux,bash,zsh,nano,vim \
           nano vim qemacs gparted partimage fsarchiver rsync \
           networkmanager netctl openvpn wget curl sqlite

# Язык интерфейса
LANGUAGES = en,ru

# Тип загрузчика
BOOT_LOADER = grub2,isolinux

# Формат ISO
ISO_FORMAT = x86_64
```

### Выбор компонентов

Выберите только необходимые компоненты для уменьшения размера образа:
* **Базовый**: coreutils, bash, zsh, nano
* **Файловая система**: gparted, partimage, fsarchiver
* **Сеть**: networkmanager, netctl, openvpn
* **Графика**: xorg-server, xterm

## Сборка squashfs-образа

### Компиляция базовой системы

Соберите основную систему в образ squashfs:
```bash
cd sysresccd
make build
```

Это создаст файл `sysresccd.img` — образ основной системы.

### Компиляция с конкретными пакетами

Для сборки с определёнными пакетами используйте опцию PACKAGES:
```bash
make build PACKAGES="coreutils,grep,findutils,networkmanager"
```

### Сборка для конкретных архитектур

Для x86_64:
```bash
make build ARCH=x86_64
```

Для i386:
```bash
make build ARCH=i386
```

## Создание ISO-образа

### Создание ISO с помощью xorriso

Создайте ISO-образ из squashfs образа:
```bash
cd iso
xorriso -as mkisofs \
  -o ../systemrescue-x.y.z.iso \
  -V "SystemRescue x.y.z" \
  -b boot/grub/grub.cfg \
  -no-emul-boot \
  -boot-load-size 4 \
  -boot_offset 2048 \
  ../sysresccd/sysresccd.img
```

### Создание с ISO9660 уровнями

Для лучшей совместимости используйте уровни записи:
```bash
xorriso -as mkisofs \
  -o systemrescue-x.y.z.iso \
  -V "SystemRescue x.y.z" \
  -b boot/grub/grub.cfg \
  -no-emul-boot \
  -boot-load-size 4 \
  -boot_offset 2048 \
  -joliet -rock \
  -rpx \
  ../sysresccd/sysresccd.img
```

### Создание с загрузчиком Syslinux

Для использования Syslinux вместо GRUB:
```bash
xorriso -as mkisofs \
  -o systemrescue-x.y.z.iso \
  -b boot/isolinux/isolinux.bin \
  -c boot/isolinux/boot.cat \
  -no-emul-boot \
  -boot-load-size 4 \
  -boot_offset 32 \
  ../sysresccd/sysresccd.img
```

## Сборка дополнительных компонентов

### Сборка скриптов autorun

Соберите скрипты для автоматического запуска:
```bash
cd sysresccd/autorun
make build
```

### Сборка графической среды

Соберите графическую среду с X.Org:
```bash
cd sysresccd/gui
make build
```

## Установка на диск (опционально)

### Установка GRUB2 на диск

Установите GRUB2 на загрузочный раздел:
```bash
grub-install /dev/sdX
```

### Настройка loopback для GRUB2

Создайте конфигурационный файл для загрузки с образа:
```bash
mkdir -p /boot/grub2/loopback
cat > /boot/grub2/loopback.cfg << 'EOF'
set prefix=(hd0,msdos1)/boot/grub2
insmod loop_dev
insmod iso9660
insmod ext2
search --no-floppy --fs-uuid --set=root YOUR_UUID
loopback loop /sysresccd/sysresccd.img
linux (loop)/boot/x86_64/vmlinuz archisobasedir=sysresccd setkmap=us
initrd (loop)/boot/x86_64/sysresccd.img
EOF
```

## Тестирование сборки

### Проверка образа ISO

Проверьте целостность созданного ISO:
```bash
sha256sum systemrescue-x.y.z.iso
```

Загрузитесь с созданного ISO и проверьте что система работает корректно.

### Проверка размера

Проверьте размер образов:
```bash
ls -lh sysresccd/sysresccd.img
ls -lh ../systemrescue-x.y.z.iso
```

Типичные размеры:
* squashfs образ: 200-500 МБ (зависит от выбранных пакетов)
* ISO образ: 300-800 МБ (включая загрузчик и резервные копии)

## Публикация сборки

### Создание подписанного образа

Для публичной публикации создайте подписанный образ:
```bash
gpg --sign systemrescue-x.y.z.iso
```

### Обновление документации

Обновите файлы документации с информацией о сборке:
* `manual_Building_SystemRescue_from_Source_en.md`
* `README.md`

## Частые проблемы и решения

### Проблема: Ошибка компиляции gcc

Решение: Убедитесь что установлены linux-headers:
```bash
pacman -S linux-headers-linux-version
```

### Проблема: Недостаточно места на диске

Решение: Очистите временные файлы сборки:
```bash
make clean
rm -rf *.o *~
```

### Проблема: Файлы squashfs повреждены

Решение: Пересоздайте образ с проверкой целостности:
```bash
mksquashfs sysresccd/ sysresccd.img -comp xz -b 1024k
```

## Заключение

Сборка SystemRescue из исходного кода — мощный инструмент для кастомизации системы. Следуйте инструкциям выше и адаптируйте конфигурацию под ваши нужды. Не забудьте проверить созданную систему перед публикацией.
