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
bash -x /media/target/slax/bootinst.sh
mkdir -p /media/target/EFI/Boot/
cp /usr/lib/SYSLINUX.EFI/efi64/syslinux.efi /media/target/EFI/Boot/bootx64.efi
cd /usr/lib/syslinux/modules/efi64/
cp syslinux.efi ldlinux.e64 menu.c32 libcom32.c32 libutil.c32 vesamenu.c32 /media/target/EFI/Boot
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