# Создание backing store

## Обзор (Overview)

SystemRescue based на read-only compressed file system. Все файлы системы хранятся в большом squashfs file system image файле. Как следствие, **изменения которые вы делаете на системе не сохраняются**, и они теряются когда вы reboot (за исключением того что вы делаете на других file systems которые вы можете mount).

Root file system в SystemRescue — overlayfs начиная с версии 6.0. Следовательно **все изменения сделанные на системных файлах разрешены и хранятся в memory**. Это позволяет вам изменить системный файл, например вы можете заменить программу вашей собственной версией, или вы можете сделать configuration changes в приложениях таких как Firefox.

Пользователи которые хотят сохранить свои изменения в системных файлах могут создать custom SystemRescue media. Это очень удобно когда вы хотите добавлять новые программы к системе, но это не комфортно если вам часто нужно менять файлы в системе. Вы не хотите создавать новую customized версию каждый день.

Вот почему SystemRescue предоставляет backing-store feature. A backing-store — это file system хранящийся на USB-stick или на hard drive, который содержит все файлы системы которые были изменены. Modifications сохраняются в backing-store каждый раз когда вы edit файл, когда вы create новый directory, или когда система пишет или удаляет файл по любой другой причине. Как следствие, это позволяет вам сохранять ваши configuration changes: вы можете добавить ваши bookmarks и extensions к Firefox, и они всё ещё будут там когда вы reboot SystemRescue, طالما тот же backing-store загружен.

Backing stores не совместимы между SystemRescueCd-5.x и SystemRescue начиная с версии 6.0 поскольку они используют разные union file systems (aufs vs overlayfs).

## Использование backing store с SystemRescue-6.x и новее

Вам нужно иметь Linux file system где SystemRescue разрешено хранить свои изменения. Это может быть любой Linux file system такой как ext4, xfs или btrfs. Вы можете либо создать dedicated file system для хранения этих изменений или вы можете reuse существующий file system поскольку изменения будут изолированы в directory. Вам нужно предоставить SystemRescue опцию на boot command line или в <a href="/manual/Configuring_SystemRescue/">YAML configuration</a> configuration file чтобы она могла identify file system где хранить эти изменения. Recommended method — specify file system label но это не единственный способ.

Затем вам нужно запустить SystemRescue с опцией <code>cow_label=xxxx</code> где <code>xxxx</code> — это label Linux file system где вы хотите хранить изменения. Например вы используете <code>cow_label=boot</code> если ваш <code>/boot</code> file system labelled <code>boot</code>. Это similar к <code>cow_device=/dev/disk/by-label/boot</code>.

Как установить file system label зависит от используемого filesystem. Для ext4 используйте <code>tune2fs -L &lt;newlabel&gt; &lt;device&gt;</code>, для xfs <code>xfs_admin -L &lt;newlabel&gt; &lt;device&gt;</code> и <code>btrfs filesystem label &lt;device&gt; &lt;newlabel&gt;</code> для btrfs.

Изменения по умолчанию будут храниться в directory named <code>persistent_${archisolabel}/${arch}</code>. Так например <code>persistent_RESCUE904/x86_64</code>. Вы можете override этот default используя опцию <code>cow_directory=xxxx</code>.

Смотрите <a href="/manual/Booting_SystemRescue/">Booting SystemRescue</a> для больше деталей о boot options supported в SystemRescue.

---

**Дата перевода:** 2026-11-03  
**Статус:** Переведено (1 документ)  
**Источник:** https://www.system-rescue.org/manual/Creating_a_backing_store/
