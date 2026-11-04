# Установка дополнительных пакетов из AUR с помощью yay

Arch Linux предоставляет software packages с different levels of support. Core packages получают comprehensive testing и предоставляются как binary packages. Эти <a href="/manual/Installing_packages_with_pacman/">могут напрямую устанавливаться с pacman</a>.

Более niche software offered в <a href="https://wiki.archlinux.org/title/Arch_User_Repository">Arch User Repository (AUR)</a>. AUR only содержит instructions (<code>PKGBUILD</code> файл) как build package, user должен compile их packages themselves. Это может быть automated с helper programs like <code>yay</code>.

## AUR package list

Search the AUR package list: <a href="https://aur.archlinux.org/packages">https://aur.archlinux.org/packages</a>

## yay AUR helper

<a href="https://github.com/Jguer/yay">yay</a> — это AUR helper program который может search для packages в AUR, download, build и install их от commandline. Он включён в SystemRescue начиная с версии 9.05.

## Подготовка SystemRescue

SystemRescue просто comes с software который directly related к goals diagnosing и fixing system problems. Чтобы keep it lightweight и fast, components required для compile programs stripped out. Перед тем как package от AUR может быть compiled, эти parts должны быть downloaded и (re-)installed от Arch Linux online repositories.

Installing эти packages займёт over a gigabyte в Copy-on-Write (CoW) storage. CoW data по умолчанию stored в RAM и по умолчанию allowed to grow up to 25% available RAM (<code>cow_spacesize=</code> <a href="/manual/Booting_SystemRescue/">boot option</a>). Это может быстро become limitation. Из-за этого recommended store Copy-on-Write (CoW) data на hard drive вместо того. Смотрите <a href="/manual/Creating_a_backing_store/">Creating a backing-store to keep your modifications</a> для деталей.

Для подготовки SystemRescue вызовите: <code>yay-prepare</code>.

## Building packages

Search для packages с <code>yay &lt;Search Term&gt;</code>.

Если один или более packages найдены, вы можете directly choose чтобы download, build и install их.

Будьте осторожны с этими шагами однако: Packages в AUR не проходят peer review process. Так что uploading malicious packages possible. Они могут execute arbitrary commands во время build и install. Так что review package source перед building и watch for comments и votes на AUR website.

## Build Details

Building packages как root user not advised и yay prevents это. SystemRescue создаёт dedicated user <code>yay</code> когда <code>yay-prepare</code> is run. The <code>yay</code> command routed через wrapper который автоматически changes к <code>yay</code> user когда running <code>yay</code> command.

Когда build finished, вы можете find packages как <code>*.pkg.tar.zst</code> files below <code>/home/yay/.cache/yay/</code>. От there вы можете save и reinstall их позже, например с <a href="/Modules/">SystemRescue Module (SRM)</a>.

Вы можете install local <code>*.pkg.tar.zst</code> файл с command <code>pacman -U &lt;filename&gt;</code>.

---

**Дата перевода:** 2026-11-03  
**Статус:** Переведено (1 документ)  
**Источник:** https://www.system-rescue.org/manual/Installing_packages_from_AUR/
