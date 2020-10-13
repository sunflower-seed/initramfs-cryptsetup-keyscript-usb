# initramfs-cryptsetup-keyscript-usb
A custom script to unlock an encrypted [LUKS](https://en.wikipedia.org/wiki/Linux_Unified_Key_Setup) volume using a usb key or mmc storage device.
If the key is missing or the decryption process fails, the script will prompt for the key or to type the password manually.

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
sudo shred -f -z tempKeyFile.bin 
```
3. Fill the `decryptkeydevice.conf` File with the details of the key you created in Step 1 and 2 and copy it to
```
# /etc/decryptkeydevice/decryptkeydevice.conf
# ID(s) of the USB/MMC key(s) for decryption (separated by blanks)
# as listed in /dev/disk/by-id/
DECRYPTKEYDEVICE_DISKID="mmc-XXX_0x0AAABBBCCCDDD usb-XyzFlash_XYZDFGHIJK_XXYYZZ00AA-0:0"
# blocksize usually 512 is OK
DECRYPTKEYDEVICE_BLOCKSIZE="512"
# start of key information on keydevice DECRYPTKEYDEVICE_BLOCKSIZE * DECRYPTKEYDEVICE_SKIPBLOCKS
DECRYPTKEYDEVICE_SKIPBLOCKS="1"
# length of key information on keydevice DECRYPTKEYDEVICE_BLOCKSIZE * DECRYPTKEYDEVICE_READBLOCKS
DECRYPTKEYDEVICE_READBLOCKS="4"
```

4. Add path to the keyscript to `/etc/crypttab` and make it executeable
```
# /etc/crypttab
# X is the device number and Y is he UUID of the encrypted volume
sdaX_crypt UUID=Y none luks,keyscript=/etc/decryptkeydevice/decryptkeydevice_keyscript.sh

# make the script executable
sudo chmod +x /etc/decryptkeydevice/decryptkeydevice_keyscript.sh 
```

5. Copy `decryptkeydevice.hook` to `/etc/initramfs-tools/hooks` and make it executable
```
sudo chmod +x /etc/initramfs-tools/hooks/decryptkeydevice.hook
```

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
