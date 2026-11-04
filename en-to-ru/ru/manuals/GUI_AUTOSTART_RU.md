# Запуск программ когда графический десктоп появляется: gui_autostart

## Обзор

SystemRescue содержит X.Org графический десктоп который может быть запущен либо используя [boot options](/manual/Booting_SystemRescue/) `dostartx` или `dovnc` или выполняя `startx` на командной строке. Вы можете автоматически запускать программы как только графический десктоп запущен.

## Конфигурация

Конфигурация для этого делается исключительно через записи в [YAML configuration](/manual/Configuring_SystemRescue/) используя область действия `gui_autostart`. Ключи в map/dictionary (100 и 200 в примере) — это имена для каждой программы. Имена используются для `.desktop` файлов которые создаются для реализации этого:

```yaml
---
gui_autostart:
    100:
        desktop: "/usr/share/applications/thunar.desktop"
    200:
        exec: "bash -c \"read -p 'press any key'"
        terminal: true
```

## desktop

Если уже есть `.desktop` файл (см. ниже) для программы которую вы хотите запустить, вы можете использовать ключ `desktop` чтобы указать на этот файл.

Использование ключа `desktop` mutually exclusive с ключом `exec`.

## exec

Если нет `.desktop` файла для вашей программы или он отсутствует вещи типа правильных аргументов, вы можете использовать ключ `exec` чтобы выполнить произвольную команду. Содержимое берётся как есть и записывается в ключ `Exec` созданного `.desktop` файла. Смотрите [документацию ключа Exec](https://specifications.freedesktop.org/desktop-entry-spec/desktop-entry-spec-latest.html#exec-variables) для деталей.

Когда используете `exec` вы также можете установить `terminal` чтобы программа работала в терминальном окне (опция `Terminal` в `.desktop` файле).

Использование ключа `exec` mutually exclusive с ключом `desktop`.

## Implementation and relevant standards

Функция реализована созданием .dektop файлов в директории ~/.config/autostart, как описано:

* [Overview in the Arch Wiki](https://wiki.archlinux.org/title/Desktop_entries)
* [Desktop Application Autostart Specification](https://specifications.freedesktop.org/autostart-spec/autostart-spec-latest.html)
* [Desktop Entry Specification](https://specifications.freedesktop.org/desktop-entry-spec/desktop-entry-spec-latest.html)

---

**Дата перевода:** 2026-11-03  
**Статус:** Переведено (1 документ)  
**Источник:** https://www.system-rescue.org/manual/gui_autostart_Start_programs_on_graphical_desktop/
