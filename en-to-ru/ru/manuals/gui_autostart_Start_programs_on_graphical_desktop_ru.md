+++
title = "Запуск программ при появлении графического рабочего стола: gui_autostart"
draft = false
---

## Обзор

SystemRescue содержит графический рабочий стол X.Org, который может быть запущен либо через опции загрузки `dostartx` или `dovnc` [Booting_SystemRescue/](/manual/Booting_SystemRescue/) или выполнение `startx` в командной строке. Можно автоматически запускать программы как только графический рабочий стол запущен.

## Конфигурация

Конфигурация для этого делается исключительно через записи в [YAML-конфигурации](/manual/Configuring_SystemRescue/) используя область `gui_autostart`. Ключи в map/словаре (100 и 200 в примере) — это имена для каждой программы. Имена используются для `.desktop` файлов, которые создаются для реализации этого:

```
---
gui_autostart:
    100:
        desktop: "/usr/share/applications/thunar.desktop"
    200:
        exec: "bash -c \"read -p 'press any key'\""
        terminal: true
```

## desktop

Если уже есть `.desktop` файл (см. ниже) для программы которую хотите запустить, можно использовать ключ `desktop` чтобы указать на этот файл.

Использование ключа `desktop` взаимно исключает использование ключа `exec`.

## exec

Если нет `.desktop` файла для вашей программы или он не имеет вещей типа правильных аргументов, можно использовать ключ `exec` для выполнения произвольной команды. Содержание берётся как есть и записывается в ключ `Exec` сгенерированного `.desktop` файла. См. [документацию ключа Exec](https://specifications.freedesktop.org/desktop-entry-spec/desktop-entry-spec-latest.html#exec-variables) для деталей.

Когда используете `exec` также можно установить `terminal` чтобы программа работала в терминальном окне (`Terminal` опция в `.desktop` файле).

Использование ключа `exec` взаимно исключает использование ключа `desktop`.

## Реализация и соответствующие стандарты

Функция реализована созданием .desktop файлов в директории ~/.config/autostart, как описано:
  * [Обзор в Arch Wiki](https://wiki.archlinux.org/title/Desktop_entries)
  * [Desktop Application Autostart Specification](https://specifications.freedesktop.org/autostart-spec/autostart-spec-latest.html)
  * [Desktop Entry Specification](https://specifications.freedesktop.org/desktop-entry-spec/desktop-entry-spec-latest.html)
