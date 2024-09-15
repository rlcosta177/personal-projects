### Partitioning

I am going to use GPT partition table with EFI system type and the GRUB boot loader.
This will change from system to system, so follow these steps to check your system's BIOS configuration and change them accordingly:
  [https://pastebin.com/KYuj0gGD]

1. My system has an SSD(/dev/nvme0n1) and an HDD(/dev/sda)

2. Partition Schema:
   - SDD will contain the boot loader('/boot', 512M) and the root('/', the remaining size left by the /boot) partitions
   - HDD will contain the home('/home') partition, which will be the size of the whole disk

3. Partitioning the SSD(nvme0n1):
   - 'fdisk /dev/nvme0n1'
   - Boot partition:
     - 'g' to choose the GPT partitioning type
     - 'p' to create a partition
     - choose (1) for the Boot partition
     - leave the first Sector as default(usually 2048)
     - for the last Sector, choose +512M OR +1G. Depends on your system, but 512M should be enough on most cases
   - Root partition:
     - 'g' to choose the GPT partitioning type
     - 'p' to create a partition
     - choose (2) for the Root partition
     - leave the first Sector as default
     - choose the default value, which should be the rest of the disk's size

4. Partitioning the HDD(sda)
   - 'fdisk /dev/sda'
   - Home partition:
     - 'g' to choose the GPT partitioning type
     - 'p' to create a partition
     - choose (1) for the Home partition
     - choose the default first sector
     - choose the default last sector
    
5. Check the partitions:
   - 'lsblk -f'
   - 'fdisk -l'

---

### Format the partitions:

1. Format the Boot partition(nvme0n1p1) as  FAT32
   - 'mkfs.fat -F32 /dev/nvme0n1p1'

2. Format the Root partition(nvme0n1p2) as ext4
   - 'mkfs.ext4 /dev/nvme0n1p2'
  
3. Format the Home partition(sda1) as ext4
   - 'mkfs.ext4 /dev/sda1'
  
4. Check the partitions:
   - 'lsblk -f'
   - 'fdisk -l'
  
---
  
### Mount the partitions

Arch linux's instalation requires us to mount the partitions in the /mnt directory.
The /mnt will later on become the root partition, replacing the original root directory.

1. Mount the root partition(/dev/nvme0n1p2)
   - 'mount /dev/nvme0n1p2 /mnt'

2. Mount the boot partition(/dev/nvme0n1p1)
   - 'mount --mkdir /dev/nvme0n1p1 /mnt/boot'

3. Mount the home partition(/dev/sda1)
   - 'mount --mkdir /dev/sda1 /mnt/home'

---

### Install the base system

This is where we replace the current root directory('/') with the /mnt directory(the one where we mounted the partitions above)


1. Install essencial packages
   - 'pacstrap -K /mnt base base-devel linux linux-firmware e2fsprogs dhcpcd networkmanager sof-firmware git man-db man-pages nano'

2. Generate the file system table && check if it is correct
   - 'genfstab -U /mnt >> /mnt/etc/fstab'
   - cat /mnt/etc/fstab
  
3. Chroot into the new system
   - 'arch-chroot /mnt'
  
4. Set the time-zone
   - 'ln -sf /usr/share/zoneinfo/Atlanta/Azores /etc/localtime'
   - 'hwclock --systohc'
   - 'hwclock --show'
  
5. Set locale
   - nano /etc/locale.gen (Uncomment your preferred locale. e.g. 'en_US.UTF-8 UTF')
   - save the file and run 'locale-gen'
   - 'echo "LANG=en_US.UTF-8" > /etc/locale.conf'
  
6. Set hostname
   - 'echo "myhostname" > /etc/hostname'
   - e.g. 'echo ronaldoSUII > /etc/hostname'
   - 'nano /etc/hosts':
      ```bash
      127.0.0.1   localhost
      ::1         localhost
      127.0.1.1   myhostname.localdomain myhostname
      ```

7. Set root password
   - passwd

8. Install the Boot Loader
   - GRUB & EFI support: 'pacman -S grub efibootmgr'
   - 'grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=GRUB'
   - 'grub-mkconfig -o /boot/grub/grub.cfg'
  
9. Exit chroot and reboot
    - 'exit'
    - 'umount -R /mnt' (unmounting is necessary to avoid conflicts when restarting the system)
    - 'shutdown' -> remove the usb stick -> boot the system back up










