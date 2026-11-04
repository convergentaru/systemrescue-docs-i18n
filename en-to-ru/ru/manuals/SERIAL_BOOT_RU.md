# Загрузка на serial console

## Serial console

Serial консоли вместо обычной видеокарты с VGA или HDMI/DP выходом распространены на embedded системах, но также используются для удалённого управления на серверах.

Для работы serial консоли нужно знать baud rate и параметры используемые а также внутренний порт номер используемый. SystemRescue использует наиболее распространённую setup по умолчанию, что первый serial порт (**ttyS0** или COM1:) с **baud rate 115200, 8 Bits, no parity, 1 stop bit**.

## Bootmanager

Оба boot manager используемые (syslinux и grub) позволяют использовать serial console полностью параллельно обычной графической консоли без приоритизации одного над другим. Так serial console всегда активируется с параметрами заданными в config файлах (см. ниже).

## Booting a kernel

Linux kernel также позволяет использовать несколько консолей параллельно, они все конфигурируются с `console=` boot commandline option. Но в отличие от boot managers, эти имеют priority и только один с highest priority получает boot сообщения и usable во время initramfs phase.

Так serial console не может всегда активироваться параллельно, но имеет dedicated boot menu option вместо этого.

## Changing parameters

Когда дефолтные параметры SystemRescue не подходят вашей системе, вы должны адаптировать их в configuration files prior to booting. Вы должны адаптировать эти файлы на iso-Image SystemRescue:

**`/boot/grub/grubsrcd.cfg`**:

```
# enable serial console with common settings (ttyS0, 115200 Baud, 8n1)
# this works in parallel to regular console
serial --unit=0 --speed=115200 --word=8 --parity=no --stop=1
terminal_input --append serial
terminal_output --append serial

[...]

menuentry "Boot SystemRescue with serial console (ttyS0,115200n8)" {
        set gfxpayload=keep
        linux /sysresccd/boot/x86_64/vmlinuz archisobasedir=sysresccd archisolabel=RESCUE905 iomem=relaxed console=tty0 console=ttyS0,115200n8
        initrd /sysresccd/boot/intel_ucode.img /sysresccd/boot/amd_ucode.img /sysresccd/boot/x86_64/sysresccd.img
}
```

**`/sysresccd/boot/syslinux/sysresccd_head.cfg`**:

```
SERIAL 0 115200
```

**`/sysresccd/boot/syslinux/sysresccd_sys.cfg`**:

```
LABEL sysresccd-serial
TEXT HELP
Use a serial console.
ENDTEXT
MENU LABEL Boot SystemRescue with serial console (ttyS0,115200n8)
LINUX boot/x86_64/vmlinuz
INITRD boot/intel_ucode.img,boot/amd_ucode.img,boot/x86_64/sysresccd.img
APPEND archisobasedir=sysresccd archisolabel=RESCUE905 iomem=relaxed console=tty0 console=ttyS0,115200n8
```

Для изменения этих файлов, используйте либо [sysrescue-customize](/scripts/sysrescue-customize/) чтобы изменить iso image или автоматизировать эти изменения или [write SystemRescue to USB media](/Installing-SystemRescue-on-a-USB-memory-stick/) способом который создаёт writeable image, как Rufus или sysrescue-usbwriter, а затем делайте изменения с text editor.

---

**Дата перевода:** 2026-11-03  
**Статус:** Переведено (1 документ)  
**Источник:** https://www.system-rescue.org/manual/Booting_on_a_serial_console/
