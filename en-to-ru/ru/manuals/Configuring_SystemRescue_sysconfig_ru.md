+++
title = "Настройка SystemRescue: область sysconfig"
draft = false
---

Эта глава объясняет опции, доступные в области `sysconfig` файлов [YAML-конфигурации](/manual/Configuring_SystemRescue/) для System Rescue. См. там для деталей где размещать YAML-файлы и как они интерпретируются.

## Синтаксис

```
---
sysconfig:
    authorized_keys:
        "user@example.com": "ssh-rsa AAAAB3NzaC1...kQoVPcn3jpgywi/twXcOw=="
        "other-user@example.com": "no-port-forwarding ssh-rsa ZDWo0UmISKEn...dAq33PUQh"
        "ca@example.org": "cert-authority,principals=\"bob,joe\" ssh-rsa AAAAB3NzaC1y...Zhk0="

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
```
