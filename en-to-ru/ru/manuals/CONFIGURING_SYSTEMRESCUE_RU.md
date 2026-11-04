# Конфигурация SystemRescue

## Обзор (Overview)

SystemRescue предоставляет опции которые позволяют пользователям изменить способ его работы. Например, есть опции для контроля какой клавиатуры и раскладки использовать, должен ли система запускаться из памяти или с устройства загрузки, автоматически ли запускать графическую среду, и так далее. Вы можете следовать ссылке чтобы увидеть список <a href="/manual/Booting_SystemRescue/">опций загрузки</a> которые поддерживаются на командной строке.

Эти опции задаются на командной строке загрузки. Чтобы изменить эти опции, вы можете либо вручную редактировать командную строку загрузки от загрузчика в runtime, или вы можете сделать изменения постоянными редактируя конфигурационные файлы загрузчиков, которые либо isolinux если вы запускаете в BIOS режиме или Grub если вы запускаете в UEFI режиме.

## Функции конфигурации (Configuration features)

Начиная с SystemRescue версии 9.00 есть другой способ конфигурировать SystemRescue. Вы теперь можете редактировать YAML файлы на boot device для конфигурирования его. Этот способ конфигурирования был introduced чтобы сделать лёгким делать persistent configuration changes, и эти опции эффективны как для BIOS режима так и для UEFI режима.

SystemRescue конфигурационные YAML файлы расположены в <code>sysrescue.d</code> папке расположенной на boot device.

SystemRescue приходит с default YAML конфигурационным файлом, так что он предоставляет хороший пример такого файла, который вы можете использовать как стартовую точку. Вы можете редактировать его чтобы заменить опции своими предпочтениями.

## Области (Scopes)

Эти yaml конфигурационные файлы поддерживают multiple scopes. Главный scope называется <code>global</code> и содержит general configuration entries. Nearly все entries в global scope имеют boot commandline options того же имени и документированы в <a href="/manual/Booting_SystemRescue/">Boot options</a>.

<code>autorun</code> scope используется для определения configuration entries которые используются <a href="/manual/Run_your_own_scripts_with_autorun/">autorun</a> и документированы там.

<code>gui_autostart</code> scope используется для определения configuration entries которые используются <a href="/manual/gui_autostart_Start_programs_on_graphical_desktop/">gui_autostart</a> и документированы там.

<code>autoterminal</code> scope используется для определения configuration entries которые используются <a href="/manual/autoterminal_scripts_on_virtual_terminal/">autoterminal</a> и документированы там.

<code>sysconfig</code> scope используется для определения certain aspects system configuration. Вы можете найти больше информации об entries на <a href="/manual/Configuring_SystemRescue_sysconfig/">sysconfig</a>.

## Configuration entries (Конфигурационные записи)

Ниже пример valid yaml конфигурационного файла. В этом примере есть entries как в <code>global</code>, <code>autorun</code>, и <code>sysconfig</code> scopes. Опция <code>copytoram</code> enabled так что система полностью copied в memory при boot времени, опция <code>checksum</code> также enabled так что система проверяет её integrity при boot времени, а также опция <code>dostartx</code> чтобы автоматически запускать графическую среду. Обе опции <code>nofirewall</code> и <code>loadsrm</code> оставлены disabled, так что firewall не будет выключен и SRM модули не будут загружены. Опция <code>setkmap</code> используется для конфигурировать french keyboard layout. Все эти general options belong к <code>global</code> scope.

```yaml
---
global:
    copytoram: true
    checksum: true
    nofirewall: false
    loadsrm: false
    late_load_srm: "https://example.com/myconfig.srm"
    setkmap: "fr-latin1"
    dostartx: true
    dovnc: false
    rootshell: "/bin/bash"
    rootcryptpass: "$6$Y.AolXkpG/Js2Zqx$z7J893qtB7jKn3z39ucbgvpkJ6wTrJ8N0CBVr5cJ.uXugGTMTSjMI7qsSTu4UTFGGKpGyEG/BnYNRE6oZFO4b0"
    rootpass: "MyRootPassword123"
    vncpass: "MyVncPassword456"

autorun:
    ar_disable: false
    ar_nowait: false
    ar_nodel: false
    ar_attempts: 1
    ar_ignorefail: false
    ar_suffixes: "0,1,2,3,4,5"

autoterminal:
    tty2: "/usr/bin/tmux"
    
sysconfig:
    ca-trust:
        example-ca: |
            -----BEGIN CERTIFICATE-----
            MIIDlTCCAn2gAwIBAgIUbB4K7H53E3spHfMtSb0To+Fyb3wwDQYJKoZIhvcNAQEL
            BQAwWjELMAkGA1UEBhMCWFgxFTATBgNVBAcMDERlZmF1bHQgQ2l0eTEcMBoGA1UE
            [...]
            VtbLuXNBNjfcAk1xqTb1j9dMeHDZKV4Imr0W3qfsHnWFqihxGyKJ79Qb2bL1Kquc
            vgI/6+yHyDlw
            -----END CERTIFICATE-----
```

## Статус этой функции (Status of this feature)

На этом этапе только некоторые опции supported в конфигурационном файле. План добавить support для больше опций или services в future. Также вы можете воспользоваться этим механизмом и использовать эти yaml конфигурационные файлы чтобы конфигурировать ваши собственные скрипты которые выполняются от SystemRescue. Вы должны создавать новые scopes в ваших конфигурационных файлах если вы планируете делать это.

Эти features relatively recent, так что рекомендуется использовать latest версию SystemRescue чтобы воспользоваться latest features и bug fixes related к способу support для конфигурирования implemented.

Способ YAML конфигурационных файлов загружен изменился с SystemRescue Version 9.03. Версии 9.00 to 9.02 только implement partial merging и когда опция <code>sysrescuecfg</code> использовалась, только файлы данных были загружены.

## Мerging и порядок конфигурационных файлов (Configuration file merging and order)

Чтобы able configure SystemRescue используя YAML конфигурационные файлы, вы либо edit существующий YAML файл located в <code>sysrescue.d</code> на boot device, или create additional YAML files в той же location. Files должны иметь <code>.yaml</code> extension (не <code>.yml</code>), иначе файл будет ignored. Также убедитесь что вы следуете yaml syntax правильно. Key и values разделяются colons, не equal signs, и indentation очень important.

Система читает все файлы с <code>.yaml</code> extension located в <code>sysrescue.d</code> папке на boot device в lexicographic order. **Files загруженные позже merged с previous ones, overwriting уже configured individual options.** Когда later loaded файл содержит empty option, он removed из final config и не просто overwritten с empty value.

Файлы предоставленные SystemRescue имеют name starting с number чтобы сделать ordering более explicit. Рекомендуется следовать этой practise для additional файлов тоже.

Например если у вас есть файл <code>100-defaults.yaml</code>:

```yaml
---
global:
    nofirewall: false
    dovnc: false
    setkmap: "fr-latin1"
```

и файл <code>200-my-options.yaml</code>:

```yaml
---
global:
    nofirewall: true
    setkmap:
```

<code>nofirewall</code> option будет <code>true</code>, <code>dovnc</code> будет <code>false</code> и no config entry будет существовать для <code>setkmap</code>.

После все файлы в <code>sysrescue.d</code> dir прочитаны и merged, система looks для <code>sysrescuecfg</code> option на boot command line. Вы можете добавить больше YAML files чтобы быть loaded и merged с той опцией.

<code>sysrescuecfg</code> option позволяет использовать HTTP и HTTPS URLs для loading remote files. Когда используя HTTPS, **no certificate check is done**, как CA trust database не set up ещё во время initramfs boot phase когда YAML файлы evaluated.

Alternatively вы можете specify absolute и relative paths. Relative paths are relative к <code>sysrescue.d</code> dir на boot device. Если given path points к directory, все <code>.yaml</code> files в этом directory loaded и merged в lexicographic order. Это может быть использовано например для loading YAML files от different subdirectories <code>sysrescue.d</code>. Вы можете specify <code>sysrescuecfg</code> option multiple times на boot command line.

После YAML files loaded и merged, options на boot command line evaluated. Смотрите <a href="/manual/Booting_SystemRescue/">boot options</a>. Options на boot command line take precedence к тому что configured в YAML files.

## Storing YAML configuration files (Хранение YAML конфигурационных файлов)

The easiest method чтобы получить ваши configuration files onto SystemRescue media обычно create writable USB boot media для SystemRescue. Это может быть done с Rufus или SystemRescue USB writer <a href="/Installing-SystemRescue-on-a-USB-memory-stick/">как описано здесь</a>. Вы можете mount USB media afterwards и copy configuration files into <code>sysrescue.d</code> directory используя regular file manager.

Если вы install SystemRescue на USB stick используя либо <code>dd</code> или любой другой tool который performs physical copy ISO image, он не будет производить writable filesystem, и вы не сможете edit configuration file на device.

Если creating writable USB media не possible, вы можете modify ISO image с <a href="/scripts/sysrescue-customize/">sysrescue-customize</a>.

## Implementation details (Детали реализации)

Support для options located в YAML конфигурационном файле implemented в multiple places:

The bulk обработки configuration implemented в following script: <a href="https://gitlab.com/systemrescue/systemrescue-sources/-/blob/main/airootfs/usr/bin/sysrescue-configuration.lua">sysrescue-configuration.lua</a>. Этот script run во время initramfs boot phase и обрабатывает yaml конфигурационные файлы available на local boot device, а также options specified на boot command line, и он определяет "effective" configuration. Это single JSON файл, stored в <code>/run/archiso/config/sysrescue-effective-config.json</code>, который содержит single definition каждой supported option и позволяет various programs определить какое значение applicable для particular option, без необходимости processing все possible sources конфигурации. Multiple scripts такие как <code>sysrescue-initialize</code> и <code>sysrescue-autorun</code> используют effective configuration file чтобы определить что делать. Python scripts имеют built-in support для reading JSON files. Shell script могут использовать standard <code>jq</code> command чтобы read values из этого JSON файла.

Lua library используемая чтобы read YAML файлы и process их в JSON файл puts некоторые restrictions на supported YAML syntax и types. Например все numeric values converted к floating point by lua и written out таким образом в JSON.

Когда adding новые values рекомендуется использовать list structures очень carefully, потому что нет obvious semantics для merge их. New list values currently overwrite values в той же position когда merging multiple files. Using dictionary структуры которые затем evaluated с их keys в lexicographic order — preferred way вместо этого. Это позволяет full merging и targeted removal previous entries.

Some <a href="https://gitlab.com/systemrescue/systemrescue-sources/-/blob/main/patches/archiso-v43-07-yaml-config.patch">archiso hooks</a> read effective configuration чтобы определить values для options такие как <code>copytoram</code>, <code>checksum</code> и <code>loadsrm</code> которые должны быть использованы на early stage во время boot process, как часть initramfs.

Sysrescue-initialize script также использует effective configuration чтобы определить как система должна быть initialized, в later stage boot process. Он разделён в две части: <a href="https://gitlab.com/systemrescue/systemrescue-sources/-/blob/main/airootfs/etc/systemd/scripts/sysrescue-initialize-prenet">sysrescue-initialize-prenet</a> и <a href="https://gitlab.com/systemrescue/systemrescue-sources/-/blob/main/airootfs/etc/systemd/scripts/sysrescue-initialize-whilenet">sysrescue-initialize-whilenet</a>. Первый выполняется перед setting up networking начинается и конфигурирует вещи такие как firewalling, latter выполняется параллельно networking being set up.

---

**Дата перевода:** 2026-11-04  
**Статус:** Переведено (1 документ)  
**Источник:** https://www.system-rescue.org/manual/Configuring_SystemRescue/
