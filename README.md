# slax-efi
EFI configuration for slax

## Preparation

Download slax from https://slax.org
I've tested this with http://ftp.sh.cvut.cz/slax/Slax-9.x/slax-64bit-9.9.1.iso

## Usage
Either copy the contents of EFI to your thumb drive or boot slax and copy the files from Debian yourself.

~~~bash
thumbdrive=/dev/sdb
d=`mktemp -d`
cd $d
wget http://ftp.sh.cvut.cz/slax/Slax-9.x/slax-64bit-9.9.1.iso
test -f slax-64bit-9.9.1.iso
apt install -y parted syslinux-common syslinux-efi
dd if=/dev/zero of=${thumbdrive} bs=1M count=5
sync
/sbin/parted ${thumbdrive} mklabel msdos --script
/sbin/parted ${thumbdrive} mkpart primary 0% 100% --script
mkfs.vfat ${thumbdrive}1
mkdir /media/{source,target}
mount -o loop -t iso9660 slax-64bit-9.9.1.iso /media/source
mount -t vfat ${thumbdrive}1 /media/target
cp -axv /media/source/slax /media/target/slax
bash -x /media/target/slax/boot/bootinst.sh
mkdir -p /media/target/EFI/Boot/
cp /usr/lib/SYSLINUX.EFI/efi64/syslinux.efi /media/target/EFI/Boot/bootx64.efi
cd /usr/lib/syslinux/modules/efi64/
cp ldlinux.e64 menu.c32 libcom32.c32 libutil.c32 vesamenu.c32 /media/target/EFI/Boot
cat > /media/target/EFI/Boot/syslinux.cfg <<"SYSLINUXCFG"
TIMEOUT 30
ONTIMEOUT slax

UI vesamenu.c32
MENU TITLE Boot (EFI)

LABEL slax
      MENU LABEL Run Slax (Persistent changes)
      LINUX /slax/boot/vmlinuz
      INITRD /slax/boot/initrfs.img
      APPEND vga=normal load_ramdisk=1 prompt_ramdisk=0 rw printk.time=0 slax.flags=perch,automount
SYSLINUXCFG
~~~

> **Please Note (1):**
> *efibootmgr* can only configure EFI boot entries when you have already booted from EFI. 
> Because of that, the boot files have to reside in hardcoded directories according to UEFI standard.
> One possibility is `/EFI/Boot`

> **Please Note (2):**
> It suffices to format your thumb drive with fat32. No need to change the partition type to EF.

> **Please Note (3):**
> The thumb drive uses regular MBR partitioning!

## Files on Thumbdrive

Thumbdrive should now have the following files:
~~~
# find /media/target/
/media/target/
/media/target/slax
/media/target/slax/01-core.sb
/media/target/slax/01-firmware.sb
/media/target/slax/02-xorg.sb
/media/target/slax/03-desktop.sb
/media/target/slax/04-apps.sb
/media/target/slax/05-chromium.sb
/media/target/slax/changes
/media/target/slax/boot
/media/target/slax/boot/bootinst.bat
/media/target/slax/boot/bootinst.sh
/media/target/slax/boot/bootlogo.png
/media/target/slax/boot/extlinux.x32
/media/target/slax/boot/extlinux.x64
/media/target/slax/boot/help.txt
/media/target/slax/boot/initrfs.img
/media/target/slax/boot/isolinux.bin
/media/target/slax/boot/isolinux.boot
/media/target/slax/boot/ldlinux.c32
/media/target/slax/boot/libcom32.c32
/media/target/slax/boot/libutil.c32
/media/target/slax/boot/mbr.bin
/media/target/slax/boot/pxelinux.0
/media/target/slax/boot/runadmin.vbs
/media/target/slax/boot/samedisk.vbs
/media/target/slax/boot/syslinux.cfg
/media/target/slax/boot/syslinux.com
/media/target/slax/boot/syslinux.exe
/media/target/slax/boot/vesamenu.c32
/media/target/slax/boot/vmlinuz
/media/target/slax/boot/zblack.png
/media/target/slax/boot/ldlinux.sys
/media/target/slax/modules
/media/target/EFI
/media/target/EFI/Boot
/media/target/EFI/Boot/bootx64.efi
/media/target/EFI/Boot/ldlinux.e64
/media/target/EFI/Boot/menu.c32
/media/target/EFI/Boot/libcom32.c32
/media/target/EFI/Boot/libutil.c32
/media/target/EFI/Boot/vesamenu.c32
/media/target/EFI/Boot/syslinux.cfg
~~~

## Did it work?

Boot from the thumbdrive using EFI and check whether it worked with:

~~~
# ls -l /sys/firmware/efi/
total 0
-r--r--r--  1 root root 4096 Feb 24 21:03 config_table
drwxr-xr-x  2 root root    0 Feb 24  2020 efivars
-r--r--r--  1 root root 4096 Feb 24 21:03 fw_platform_size
-r--r--r--  1 root root 4096 Feb 24 21:03 fw_vendor
-r--r--r--  1 root root 4096 Feb 24 21:03 runtime
drwxr-xr-x  6 root root    0 Feb 24 21:03 runtime-map
-r--------  1 root root 4096 Feb 24 21:03 systab
drwxr-xr-x 27 root root    0 Feb 24  2020 vars
~~~

This directory only exists, when booting in EFI mode.
