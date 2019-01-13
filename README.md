# initramfs-cryptsetup-keyscript-usb
Keyscript for decrypting a full-encrypted luks disk using a usb/mmc storage.

If the decryption process fails you be asked for a password at boot, like usual.

## Prerequisites
A Linux distribution with an initramfs system.
If you use a complete systemd init you might want to use a [PasswordAgent](https://www.freedesktop.org/wiki/Software/systemd/PasswordAgents/) to achieve the same goal.

## Usage

1. Create a hidden "key" on your USB Memory device
- Read the start of your device partition
```
sudo fdisk -l /dev/sdb 
```
- Fill the free space with random stuff. Set count to the value of of your first partion - 2
```
sudo dd if=/dev/urandom of=/dev/sdb bs=512 seek=1 count=60 
```
2. Add the key to your LUKS Partition
```
sudo dd if=/dev/sdb bs=512 skip=1 count=4 > tempKeyFile.bin
sudo cryptsetup luksAddKey /dev/sda5 tempKeyFile.bin
sudo rm -f tempKeyFile.bin 
```
3. Fill the decryptkeydevice.conf File with the details of your key you createt in Step 1 and 2 and put it to
```
/etc/decryptkeydevice/decryptkeydevice.conf
```
4. Add path to keyscript.sh to */etc/crypttab* and make in executeable
```
sudo chmod +x /etc/decryptkeydevice/decryptkeydevice_keyscript.sh 
```
5. Copy *decryptkeydevice.hook* to
```
/etc/initramfs-tools/hooks/decryptkeydevice.hook
```
and make it executeable in the same way as described in Step 4
6. Finally Update your initramfs. If you see no warnings you should be able to reboot.
```
sudo update-initramfs -k `uname -r` -c 
```

## License
CC-BY-NC-SA 2.0 de

Due to the License of the original source.
May not be nothworthy, because the level of creativity is not high enough or the original authors provided a different license.

## Source
Original Source: https://wiki.ubuntuusers.de/System_verschl%C3%BCsseln/Entschl%C3%BCsseln_mit_einem_USB-Schl%C3%BCssel/
