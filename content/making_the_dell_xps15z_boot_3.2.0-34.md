Title: Making the dell xps15z boot 3.2.0-34
Date: 2012-02-01 10:20
Category: blog
Tags: ubuntu, xps15z

I have had some troubles with my laptop not always booting all the way into a linux desktop.
Sometimes it would stop after the grub screen, others it would work flawlessly. 
However it would never boot the first time from a power cycle.


Boot options
------------

Depending on your kernel version one of the two following fixes are needed (before 3.2.0-33 or on and later).

For 3.2.0-33 or later
----------------------

```sh
$ sudo vim /etc/default/grub
```

And modify the line 

```sh
GRUB_CMDLINE_LINUX_DEFAULT="acpi_backlight=vendor dell_laptop.backlight=0 quiet splash"
```

Save and Close the file then, try grub to update /boot/grub/grub.cfg

```sh
$ sudo update-grub
```

For 3.2.0-32 or before
-----------------------

```sh
$ sudo vim /etc/default/grub
```
And modify the line 

```sh
GRUB_CMDLINE_LINUX_DEFAULT="acpi=noirq quiet splash"
```

Save and Close the file then, try grub to update /boot/grub/grub.cfg

```sh
$ sudo update-grub
```
