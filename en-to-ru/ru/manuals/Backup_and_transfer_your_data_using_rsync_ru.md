+++
title = "Резервное копирование и передача данных с помощью rsync"
draft = false
---

## Обзор rsync

SystemRescue содержит `rsync` — мощный инструмент для резервного копирования и передачи данных. Rsync позволяет эффективно копировать файлы между локальными директориями или удалёнными серверами с поддержкой сжатия, разностного копирования и параллельной передачи.

## Установка rsync

Rsync обычно уже установлен на SystemRescue. Проверьте:
```bash
rsync --version
```

Если rsync не установлен, установите его:
```bash
pacman -S rsync
```

## Базовое использование

### Копирование локальных директорий

Самая простая форма rsync — копирование одной директории в другую:
```bash
rsync -av source/ destination/
```

Параметры:
* `-a` (archival) — архивный режим сохраняет права доступа, символы жестких ссылок, время и т.д.
* `-v` (verbose) — подробный вывод

### Разностное копирование

Rsync автоматически определяет какие файлы изменились:
```bash
# Первая передача (копирует все)
rsync -av /source/ /destination/

# Последующие передачи (только изменения)
rsync -av /source/ /destination/
```

### Сжатие при передаче

Для ускорения передачи через сеть используйте сжатие:
```bash
rsync -avz source/ destination/
```

Параметр `-z` сжимает данные во время передачи.

## Работа с удалёнными серверами

### Копирование на SSH-сервер

Для копирования на удалённый сервер используйте SSH:
```bash
rsync -avz user@remotehost:/remote/path/ /local/destination/
```

Или в обратном направлении:
```bash
rsync -avz /local/source/ user@remotehost:/remote/path/
```

### Публичные ключи для SSH

Для бесшовной работы без ввода пароля добавьте публичный ключ в `~/.ssh/authorized_keys` на удалённом сервере.

## Продвинутые опции

### Исключение файлов и директорий

Исключите файлы с помощью параметров исключения:
```bash
rsync -av --exclude='*.tmp' --exclude='.cache/' source/ destination/
```

Частые паттерны исключения:
```bash
rsync -av \
  --exclude='*.log' \
  --exclude='*.tmp' \
  --exclude='.cache/' \
  --exclude='.swp' \
  source/ destination/
```

### Переопределение прав доступа

Переопределите права доступа на целевом сервере:
```bash
rsync -av --perms source/ destination/
```

### Сохранение времени и владельца

Сохраните метки времени и владельцев файлов:
```bash
rsync -av --owner --group --times source/ destination/
```

### Блокировка проверок целостности

Используйте быстрые проверки для больших наборов данных:
```bash
rsync -av --checksum source/ destination/
```

## Резервное копирование с ротацией

### Копирование в несколько мест назначения

Создайте скрипт для резервного копирования в несколько мест:
```bash
#!/bin/bash
SOURCE="/home/user/data"
DATE=$(date +%Y%m%d)

rsync -avz --delete "$SOURCE/" "/backup/daily/$DATE/"
rsync -avz --delete "$SOURCE/" "/backup/weekly/"
```

### Использование rsync с ротацией по времени

```bash
#!/bin/bash
DATE=$(date +%Y%m%d)
DAY_OF_WEEK=$(date +%u)

# Ежедневное резервное копирование
rsync -avz --delete /source/ /backup/daily/$DATE/

# Еженедельное резервное копирование (только по воскресеньям)
if [ "$DAY_OF_WEEK" = "7" ]; then
    rsync -avz --delete /source/ /backup/weekly/
fi
```

## Работа с NTFS и Windows разделами

### Копирование данных с Windows раздела

Смонтируйте NTFS раздел:
```bash
mkdir -p /mnt/windows
mount -t ntfs3 /dev/sda1 /mnt/windows
```

Скопируйте данные пользователей:
```bash
rsync -avz \
  --exclude='pagefile.sys' \
  --exclude='hiberfil.sys' \
  --exclude='*.tmp' \
  /mnt/windows/Users/ \
  /backup/windows-users/
```

Размонтируйте после завершения:
```bash
umount /mnt/windows
```

## Резервное копирование базы данных

### MySQL/MariaDB

Резервное копирование MySQL с помощью mysqldump:
```bash
mysqldump --all-databases > /backup/mysql-$(date +%Y%m%d).sql
```

С использованием rsync для передачи:
```bash
mysqldump --all-databases | gzip | rsync -avz -e ssh - < user@remote:/backup/mysql/
```

### PostgreSQL

Резервное копирование PostgreSQL:
```bash
pg_dumpall > /backup/postgres-$(date +%Y%m%d).sql
```

## Мониторинг и логирование

### Логирование rsync операций

Создайте лог-файл для каждой операции:
```bash
rsync -avz --log-file=/var/log/rsync-backup.log /source/ /destination/
```

### Проверка успешности резервного копирования

Проверьте размер данных после резервного копирования:
```bash
du -sh /backup/latest/
```

## Заключение

Rsync — мощный инструмент для резервного копирования и передачи данных. Используйте его регулярно для создания надёжных резервных копий ваших важных данных.
