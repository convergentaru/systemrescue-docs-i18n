# Конфигурация SystemRescue: область sysconfig

## Введение

Эта глава объясняет доступные опции в области `<sysconfig>` файлов конфигурации YAML для System Rescue. См. там детали где размещать YAML файлы и как они интерпретируются.

---

## Сводка (Synopsis)

```yaml
---
sysconfig:
    authorized_keys:
        "[email protected]": "ssh-rsa AAAAB3NzaC1...kQoVPcn3jpgywi/twXcOw=="
        "[email protected]": "no-port-forwarding ssh-rsa ZDWo0UmISKEn...dAq33PUQh"
        "[email protected]": "cert-authority,principals=\"bob,joe\" ssh-rsa AAAAB3NzaC1y...Zhk0="

    bash_history:
        100: "setkmap"
        200: "reverse_ssh support.example.com 1234"

    bookmarks:
        0100:
            title: "SystemRescue"
            url: "https://www.system-rescue.org/"
        0200:
            title: "Arch Linux Package Search"
            url: "https://archlinux.org/packages/"

    ca-trust:
        example-ca: |
            -----BEGIN CERTIFICATE-----
            MIIDlTCCAn2gAwIBAgIUbB4K7H53E3spHfMtSb0To+Fyb3wwDQYJKoZIhvcNAQEL
            BQAwWjELMAkGA1UEBhMCWFgxFTATBgNVBAcMDERlZmF1bHQgQ2l0eTEcMBoGA1UE
            [...]
            VtbLuXNBNjfcAk1xqTb1j9dMeHDZKV4Imr0W3qfsHnWFqihxGyKJ79Qb2bL1Kquc
            vgI/6+yHyDlw
            -----END CERTIFICATE-----

    hosts:
        "192.168.1.1": "example.net.lan"
        "192.168.1.10": "foo.net.lan foo"

    rclone:
        config:
            arch-linux-archive:
                type: "http"
                url: "https://archive.archlinux.org/"

            my-nas:
                type: "webdav"
                url: "https://my-nas.local"
                vendor: "other"
                user: "my-login"
                pass: "9JbSJzgcQXsnV2dkzzhBR3za1e_rqixvKKp6"

    ssh_known_hosts:
        myhost.example.org: "ssh-ed25519 AAAAC3NzaC1l...JJTO48B"
        "@cert-authority *.mydomain.org": "ssh-rsa AAAAB3NzaC1y...Zhk0="

    sysctl:
        net.ipv4.ip_forward: "1"
        net.ipv4.conf.all.arp_filter: "1"

    timezone: "Europe/Berlin"

    display_scale: "disabled"
```

---

## SSH authorized_keys

Разрешает конфигурировать доверенные публичные ключи, которые разрешены для входа как пользователь `root` через SSH. Эти ключи добавляются в `/root/.ssh/authorized_keys`.

Эта опция структурирована как Mapping / словарь с полем "comment" строки authorized_keys SSH являющимся ключом, а начало записи до комментария являться значением. Поскольку поле "comment" часто используется для обозначения имени пользователя или email адреса владельца, записи упорядочены по этим именам владельцев. Если файл уже содержит запись с данным полем "comment", она не добавляется снова.

Вы можете использовать дополнительные опции для ключа как документировано sshd prepending их к типу ключа. Смотрите например опцию `no-port-forwarding` в сводке выше.

Вы можете доверять сигнатурам от SSH Certification Authority (CA) добавляя публичный ключ CA с опцией `cert-authority`. Опция `principals=` позволяет контролировать какие имена владельцев сертификатов ("principals", заданные с опцией `-n` при создании сертификата) разрешены для входа. Опция principals не имеет ничего общего с именами локальных учётных записей (например "root").

---

## Bookmarks (Закладки)

Эта опция позволяет конфигурировать закладки для установленного браузера Firefox. Это реализовано через файл `policies.json` Firefox.

Эта опция структурирована как два уровня Mapping / словарей. Ключ первого уровня используется для лексикографического упорядочивания записей. Каждая запись должна иметь ключи `title` и `url`.

Когда файл `policies.json` уже содержит запись с данным `title`, закладка не добавляется снова. Firefox наблюдался удаляющий записи с дублирующими URL.

---

## Certification Authority (CA) trust

Если вы поддерживаете один или несколько локальных Certification Authorities (CA), вы можете добавить их как доверенные якоря SystemRescue перечисляя их в секции `ca-trust` файла YAML. Вы должны дать каждому CA индивидуальное имя внутри секции `ca-trust`. Это имя будет использоваться как имя файла внутри директории `/etc/ca-certificates/trust-source/anchors/`.

Конфигурированные CA будут добавлены к доверенным якорям System Rescue по умолчанию, которые выведены из списка CA используемого Mozilla.

Эти CA в X.509 формате и используются в основном для TLS. Не путать с CA для SSH.

---

## Timezone (Часовой пояс)

Разрешает конфигурировать часовой пояс для использования. Принимает название часового пояса как определено в IANA Time Zone Database используемой большинством Linux дистрибутивов. По умолчанию "UTC".

---

## Sysctl

Разрешает кастомизировать параметры ядра через интерфейс `sysctl`. Эта опция структурирована как Mapping / словарь с ключом являющимся переменной sysctl и значением являющимся значением для установки. Используя pattern option программы sysctl не поддерживается, вы должны использовать явные имена переменных.

Рекомендуется использовать строки для значений в YAML конфиге, потому что числовые значения могут быть преобразованы к floating point во время config merge. Ядро отклонит floating point значения для большинства переменных.

---

## rclone

Разрешает конфигурировать [rclone](https://rclone.org/). Записи под ключом `sysconfig.rclone.config` записываются как секции в файл `/root/.config/rclone/rclone.conf`. Для создания секций в правильном формате рекомендуется использовать команду `rclone config` для конфигурации rclone и затем transfer данных из файла `rclone.conf` в YAML конфиг.

Обратите внимание что rclone требует все пароли быть закодированы в проприетарной легко обратимой схеме. SystemRescue просто копирует пароли и не кодирует или изменяет их, так что YAML конфиг должен содержать их уже в закодированном формате. Если хакер может прочитать YAML конфиг, эта кодировка не защитит пароли.

---

## bash_history

Разрешает пре-конфигурировать обычные команды в shell bash так что они могут быть доступны просто нажав стрелку вверх или поисканы с ctrl+r. Ключ Mapping / dict используется для лексикографического reverse-ordering записей. Reverse-ordering означает самая низкая запись покажется первой при нажатии стрелки вверх.

По умолчанию команда `setkmap` вставлена в bash_history с ключом `100` для возможности изменения keyboard mapping без необходимости печатать на возможно иностранной клавиатуре. Это упомянуто в welcome тексте показываемом при входе. Помните что welcome текст не адаптируется автоматически когда вы заменяете `setkmap` другой командой.

---

## hosts

Разрешает вручную конфигурировать hostname/IP mappings которые имеют приоритет над DNS lookups через файл /etc/hosts. Ключ Mapping / dict является IP (IPv4 или IPv6), значение hostname и, опционально, alias имена для него.

---

## ssh_known_hosts

Разрешает конфигурировать known host keys когда делают исходящие SSH соединения. Эти добавляются в `/etc/ssh/ssh_known_hosts`.

Эта опция структурирована как Mapping / словарь с полями "marker" и "hostnames" (space-separated) строки ssh_known_hosts являющимися ключом и полями "keytype", "base64-encoded key" и "comment" являться значением. Если файл уже содержит запись с данной записью она не добавляется снова.

Вы можете доверять host key сигнатурам от SSH Certification Authority (CA) добавляя публичный ключ CA с маркером `@cert-authority`. После этого маркера и пробела вы можете использовать wildcards для спецификации hostnames для которых CA сигнатуры доверены или использовать `*` для доверия всему что CA подписало.

---

## display_scale

Всё слишком маленькое чтобы быть usable в Xorg/XFCE по умолчанию на very high-resolution (aka HiDPI) экранах, таких как экран с разрешением 3840x2160. Scaling factor может быть применён вручную от XFCE display settings для того чтобы всё выглядело намного больше на таких экранах, но это требует manual intervention каждый раз когда user использует систему как конфигурация не persistent. Вот почему script был разработан для применения scaling factor автоматически когда graphical environment запускается. Script расположен в `/usr/bin/configure-display` и выполняется автоматически когда Xorg запускается благодаря `/etc/xdg/autostart/configure-display.desktop`. Поведение этого script контролируется значением `display_scale` в yaml configuration file. Default value этой опции `disabled` означает script не будет делать ничего. Это default поведение для предотвращения adverse affect script пока оно хорошо не протестировано. Вы должны установить эту опцию либо `auto` или число такому как `1.5` или `2.0` для того чтобы script фактически применял scaling factor когда Xorg запускается. Если вы используете `auto` script автоматически определит что scaling factor должен быть, based на screen resolution (в pixels) и size (в millimeters). Это поведение полезно в случае вы хотите использовать тот же System-Rescue media с той же конфигурацией на разных компьютерах которые не все имеют тот же тип экранов. Вы также можете использовать эту опцию с чисами такому как `1.5`, `2.0`, или `2.5` для того чтобы применить конкретный scaling factor. Число должно быть между `1.0` и `5.0` иначе оно будет considered как invalid. Script использует output `xrandr --query` для получения информации о экранах, и пишет logs в `/var/log/configure-display.log` в случае вы нуждаетесь troubleshoot issues.

---

**Дата перевода:** 2026-11-03  
**Статус:** Переведено (1 документ)  
**Источник:** https://www.system-rescue.org/manual/Configuring_SystemRescue_sysconfig/
