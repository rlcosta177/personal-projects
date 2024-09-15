# Arch Installation With Ethernet Connection

I haven't included the wifi connection commands, since i didnt need it at the time

### Partitioning

I will use the GPT partition table with EFI system type and the GRUB boot loader. This can vary between systems, so follow these steps to check your system's BIOS configuration and adjust accordingly:
[BIOS Configuration Guide](https://pastebin.com/KYuj0gGD)

1. **System Disks**: 
   - SSD: `/dev/nvme0n1`
   - HDD: `/dev/sda`

2. **Partition Schema**:
   - **SSD**:
     - Boot partition (`/boot`, 512M)
     - Root partition (`/`, remaining size)
   - **HDD**:
     - Home partition (`/home`, entire disk size)

3. **Partitioning the SSD (`/dev/nvme0n1`)**:
   - Run: `fdisk /dev/nvme0n1`
   - Boot partition:
     - Press `g` to select GPT
     - Press `n` to create a new partition
     - Select partition `(1)` for the Boot partition
     - First sector: press enter (default is usually 2048)
     - Last sector: enter `+512M` (or `+1G` depending on your needs)
   - Root partition:
     - Press `n` to create another partition
     - Select partition `(2)` for the Root partition
     - First sector: press enter (default)
     - Last sector: press enter to use the remaining disk space

4. **Partitioning the HDD (`/dev/sda`)**:
   - Run: `fdisk /dev/sda`
   - Home partition:
     - Press `g` to select GPT
     - Press `n` to create a new partition
     - Select partition `(1)` for the Home partition
     - First sector: press enter (default)
     - Last sector: press enter (default to use entire disk)

5. **Verify Partitions**:
   - Run: `lsblk -f`
   - Run: `fdisk -l`

---

### Formatting the Partitions

1. **Format the Boot partition** (`/dev/nvme0n1p1`) as FAT32:
   - Command: `mkfs.fat -F32 /dev/nvme0n1p1`

2. **Format the Root partition** (`/dev/nvme0n1p2`) as ext4:
   - Command: `mkfs.ext4 /dev/nvme0n1p2`

3. **Format the Home partition** (`/dev/sda1`) as ext4:
   - Command: `mkfs.ext4 /dev/sda1`

4. **Verify Partitions**:
   - Run: `lsblk -f`
   - Run: `fdisk -l`

---

### Mount the Partitions

In Arch Linux installation, we need to mount the partitions in the `/mnt` directory, which will later replace the root directory.

1. **Mount the Root partition** (`/dev/nvme0n1p2`):
   - Command: `mount /dev/nvme0n1p2 /mnt`

2. **Mount the Boot partition** (`/dev/nvme0n1p1`):
   - Command: `mount --mkdir /dev/nvme0n1p1 /mnt/boot`

3. **Mount the Home partition** (`/dev/sda1`):
   - Command: `mount --mkdir /dev/sda1 /mnt/home`

---

### Installing the Base System

Now we install the base system by replacing the current root (`/`) with the `/mnt` directory where the partitions are mounted.

1. **Install essential packages**:
   - Command: `pacstrap -K /mnt base base-devel linux linux-firmware e2fsprogs dhcpcd networkmanager sof-firmware git man-db man-pages nano`

2. **Generate the file system table** and verify:
   - Command: `genfstab -U /mnt >> /mnt/etc/fstab`
   - Check: `cat /mnt/etc/fstab`

3. **Chroot into the new system**:
   - Command: `arch-chroot /mnt`

4. **Set the time zone**:
   - Command: `ln -sf /usr/share/zoneinfo/Region/City /etc/localtime` (e.g. `ln -sf /usr/share/zoneinfo/America/New_York /etc/localtime`)
   - Run: `hwclock --systohc`
   - Verify: `hwclock --show`

5. **Set locale**:
   - Edit: `nano /etc/locale.gen` (Uncomment `en_US.UTF-8 UTF-8` or your preferred locale)
   - Save and run: `locale-gen`
   - Set language: `echo "LANG=en_US.UTF-8" > /etc/locale.conf`

6. **Set hostname**:
   - Set: `echo "myhostname" > /etc/hostname` (replace `myhostname` with your preferred name)
   - Update `/etc/hosts`:
     ```bash
     127.0.0.1   localhost
     ::1         localhost
     127.0.1.1   myhostname.localdomain myhostname
     ```

7. **Set the root password**:
   - Command: `passwd`

8. **Install the Boot Loader**:
   - Install GRUB and EFI support: `pacman -S grub efibootmgr`
   - Install GRUB: `grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=GRUB`
   - Generate GRUB config: `grub-mkconfig -o /boot/grub/grub.cfg`

9. **Exit chroot and reboot**:
   - Exit chroot: `exit`
   - Unmount all partitions: `umount -R /mnt`
   - Shutdown and remove the USB: `shutdown`
   - Boot the system back up
