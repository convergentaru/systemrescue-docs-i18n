# Сборка SystemRescue из исходного кода

## SystemRescue сам

SystemRescue основан на Arch Linux и использует патченную версию archiso для создания выпущенных .iso образов.

Вы можете найти все источники инструменты сборки и документацию сборки на [https://gitlab.com/systemrescue/systemrescue-sources](https://gitlab.com/systemrescue/systemrescue-sources)

## Arch Linux пакеты

Процесс сборки описанный выше импортирует все программы которые не разрабатываются напрямую как часть SystemRescue из репозиториев пакетов Arch Linux. Исходный код и инструменты сборки для всех этих программ доступны из Arch Linux.

Рекомендуется сначала прочитать [Arch Build System Documentation](https://wiki.archlinux.org/title/Arch_Build_System).

Инструкции сборки для всех пакетов хранятся в PKGBUILD файлах. Эти файлы и eventual патчи доступны для скачивания с [https://gitlab.archlinux.org/archlinux/packaging/packages](https://gitlab.archlinux.org/archlinux/packaging/packages)

Убедитесь что используете правильный branch и tagged версию. Если вы хотите пересобрать пакет используемый в конкретной версии SystemRescue используйте пакетную версию которая была актуальна на дату выпуска SystemRescue. Смотрите например `BUILD_ID` в `/etc/os-release` внутри SystemRescue для этого.

Когда у вас есть правильный PKGBUILD файл запустите `makepkg` чтобы собрать. По умолчанию это автоматически загрузит требуемые исходные файлы из upstream проекта. Альтернативно Arch Linux публикует эти источники на `https://archive.archlinux.org/repos/<date>/sources/`. Используйте дату выпуска как указано выше.

## systemrescue-custompkg

Несколько пакетов содержащихся в SystemRescue не доступны в бинарном виде из официальных Arch Linux репозиториев. Это например пакеты упакованные [Arch Linux AUR](https://aur.archlinux.org/) или упакованные SystemRescue.

PKGBUILD файлы патчи и документация для этих доступны на [https://gitlab.com/systemrescue/systemrescue-custompkg](https://gitlab.com/systemrescue/systemrescue-custompkg)

---

**Дата перевода:** 2026-11-03  
**Статус:** Переведено (1 документ)  
**Источник:** https://www.system-rescue.org/manual/Building_SystemRescue_from_Source/
