# Gentoo

Installation commands and some notes.

## Installation

- [Handbook](https://wiki.gentoo.org/wiki/Handbook:AMD64)
- [Quick Checklist](https://wiki.gentoo.org/wiki/Quick_Installation_Checklist)

Assuming `Gentoo` live usb for **amd64**.

### Preparation

#### LiveUSB

```
sudo su
```

```
passwd # change root password on livecd
```

```
passwd gentoo # change active user password on livecd
```

```
# Force disable system beep
xset -b && xset b off && xset b 0 0 0
```

#### Disks

Use `gparted`.

Assumptions: `GPT` partition table for `UEFI`.

| partition | filesystem | size             | description    | flags | label |
|-----------|------------|------------------|----------------|-------|-------|
| /dev/sda1 | fat32      | 512M             | Boot partition | boot, esp | boot |
| /dev/sda2 | linux-swap | ~ RAM size       | Linux swap     |       | swap |
| /dev/sda3 | ext4       | rest of the disk | Root           |       | gentoo |

```
swapon /dev/sda2
```

```
mkdir -p /mnt/gentoo && mount /dev/sda3 /mnt/gentoo
```

### Stage Tarball

Get the tarball link from the [gentoo download section](https://www.gentoo.org/downloads/)

```
TARBALL=;
cd /mnt/gentoo && \
wget $TARBALL && \
tar xpvf stage3-*.tar.xz --xattrs-include='*.*' --numeric-owner
```

### Ebuild repository

```
mkdir --parents /mnt/gentoo/etc/portage/repos.conf && \
cp /mnt/gentoo/usr/share/portage/config/repos.conf /mnt/gentoo/etc/portage/repos.conf/gentoo.conf
```

### DNS info

```
chronyd -q # sync time
```

```
cp --dereference /etc/resolv.conf /mnt/gentoo/etc/
```

### Change root

```
# mount the boot partition
mount /dev/sda1 /mnt/gentoo/boot
```

```
mount --types proc /proc /mnt/gentoo/proc && \
mount --rbind /sys /mnt/gentoo/sys && \
mount --make-rslave /mnt/gentoo/sys && \
mount --rbind /dev /mnt/gentoo/dev && \
mount --make-rslave /mnt/gentoo/dev && \
mount --bind /run /mnt/gentoo/run && \
mount --make-slave /mnt/gentoo/run
```

```
chroot /mnt/gentoo /bin/bash
```

```
mount -t tmpfs tmpfs /tmp
```

```
source /etc/profile && export PS1="(chroot) ${PS1}"
```

_Next steps are assuming work into chroot_

### Portage configuration

```
emerge-webrsync
```

Config portage with mine sample [/etc/portage/make.conf](src/make.conf)

```
# for emerge --autounmask-write option
for d in /etc/portage/package.*; do touch $d/zzz_autounmask; done
```

```
emerge --oneshot app-portage/cpuid2cpuflags && \
cd /etc/portage && \
cpuid2cpuflags > cpu_flags # merge into make.conf
```

```
# cd /mnt/gentoo/etc/portage && mirrorselect -D -s5 -o > mirrors # merge into make.conf
```

```
eselect profile list
# eselect profile set $NUMBER
```

### Configuring the system

```
ls /usr/share/zoneinfo # find timezone
# echo $Region/$City > /etc/timezone
```

```
emerge --config sys-libs/timezone-data
```

```
# grep $TARGET_LANGS /usr/share/i18n/SUPPORTED >> /etc/locale.gen
```

```
locale-gen
```

```
eselect locale list
# eselect locale set $taget_locale
```

```
env-update && source /etc/profile && export PS1="(chroot) ${PS1}"
```

Edit `/etc/fstab`

```
LABEL=boot		/boot		vfat		noauto,noatime	1 2
LABEL=swap		none		swap		sw		0 0
LABEL=gentoo		/		ext4		noatime		0 1
tmpfs    /tmp  tmpfs  size=4G,rw,nosuid,nodev,mode=1777  0 0
```

```
# echo "hostname=$hostname" > /etc/conf.d/hostname
# echo $hostname > /etc/hostname
```

### Kernel

```
echo "sys-kernel/linux-firmware linux-fw-redistributable no-source-code" >> /etc/portage/package.license && \
emerge -v sys-kernel/linux-firmware
```

```
emerge -v gentoo-sources
```

```
eselect kernel list
# eselect kernel set $CHOICE
```

```
# inspect Live system modules loaded per hardware
lspci -knn
```

```
cd /usr/src/linux && \
yes '' | make localmodconfig && \
make -j4 && make modules_install && make install
```

### Bootloader

```
echo 'GRUB_PLATFORMS="efi-64"' >> /etc/portage/make.conf && \
emerge sys-boot/grub && \
grub-install --target=x86_64-efi --efi-directory=/boot && \
grub-mkconfig -o /boot/grub/grub.cfg
```

### Finishing touches

```
emerge --ask --verbose --update --deep --newuse @world
```

```
emerge -v gentoolkit
```

- [Logging system](doc/logger.md)
- [Cron](doc/cron.md)
- [Sudo](doc/sudo.md)
- [Network management software](doc/networkmanager.md)
- [NTPD](doc/ntp.md)
- [Audio](doc/audio.md)
- [Xorg](doc/xorg.md)
- [Removable media](doc/removable_media.md)
- [Power management](doc/power_management.md)
- [Notification daemon](doc/notification_daemon.md)
- [Bash completion](doc/bash_completion.md)
- [Some recommendations](doc/recommendations.md)

Check out password policies in `/etc/security/passwdc.conf`

```
passwd # set root password
```

```
# useradd -g users -G cron,wheel,plugdev,audio,input,video,portage,usb -m ${USERNAME}
# passwd ${USERNAME}
```

### Clean up

```
exit # from chroot
```

```
cd && umount -R /mnt/gentoo
```

```
reboot
```
