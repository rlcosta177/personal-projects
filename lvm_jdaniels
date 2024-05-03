sudo apt update && sudo apt -y upgrade
sudo apt -y install lvm2 mdadm gdisk raidutils cryptsetup
lsblk

: <<'END'
    NAME         MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
    loop0          7:0    0 24.9M  1 loop /snap/amazon-ssm-agent/7628
    loop1          7:1    0 55.7M  1 loop /snap/core18/2812
    loop2          7:2    0   87M  1 loop /snap/lxd/27037
    loop3          7:3    0 63.9M  1 loop /snap/core20/2182
    loop4          7:4    0 40.4M  1 loop /snap/snapd/20671
    nvme1n1      259:0    0    4G  0 disk
    nvme2n1      259:1    0    4G  0 disk
    nvme3n1      259:2    0    4G  0 disk
    nvme0n1      259:3    0    8G  0 disk
    ├─nvme0n1p1  259:6    0  7.9G  0 part /
    ├─nvme0n1p14 259:7    0    4M  0 part
    └─nvme0n1p15 259:9    0  106M  0 part /boot/efi
    nvme4n1      259:4    0    6G  0 disk
    nvme6n1      259:5    0    6G  0 disk
    nvme8n1      259:8    0    4G  0 disk
    nvme7n1      259:10   0    6G  0 disk
    nvme5n1      259:11   0    6G  0 disk
END

for drive in 1 2 3 8 4 5 6 7; do sudo sgdisk -Z /dev/nvme${drive}n1; done
for drive in 1 2 3 8 4 5 6 7; do sudo sgdisk -n 0:0:0 -t 0:fd00 /dev/nvme${drive}n1; done

sudo mdadm --create /dev/md0 --level 6 --raid-devices 4 /dev/nvme{1,2,3,8}n1p1
sudo mdadm --create /dev/md1 --level 6 --raid-devices 4 /dev/nvme{4,5,6,7}n1p1

watch sudo mdadm --detail /dev/md0
watch sudo mdadm --detail /dev/md1

sudo cryptsetup luksFormat --hash=sha512 --key-size=512 --cipher=aes-xts-plain64 --verify-passphrase /dev/md0
sudo cryptsetup luksOpen /dev/md0 md0_crypt
sudo mkfs.ext4 /dev/mapper/md0_crypt

sudo pvcreate /dev/md1
sudo vgcreate vg1 /dev/md1
sudo lvcreate -n lv1 -l +100%FREE vg1
sudo cryptsetup luksFormat --hash=sha512 --key-size=512 --cipher=aes-xts-plain64 --verify-passphrase /dev/vg1/lv1
sudo cryptsetup luksOpen /dev/vg1/lv1 lv1_crypt
sudo mkfs.xfs /dev/mapper/lv1_crypt

sudo blkid | grep cryp | sudo tee -a /etc/crypttab
sudo blkid | grep cryp | sudo tee -a /etc/fstab

: <<'END'
    ubuntu@ip-172-31-15-182:~$ sudo blkid | grep cryp
    /dev/md0: UUID="627322a8-9c36-4aa4-ba0f-bba122252f8f" TYPE="crypto_LUKS"
    /dev/mapper/vg1-lv1: UUID="d441aa85-38e4-414e-bf93-2d6334997b7e" TYPE="crypto_LUKS"
    /dev/mapper/lv1_crypt: UUID="721b459c-921c-462e-8835-132e64c28921" BLOCK_SIZE="512" TYPE="xfs"
    /dev/mapper/md0_crypt: UUID="cbf994f5-673a-493a-b254-7cf6bee075c5" BLOCK_SIZE="4096" TYPE="ext4"
END

sudo nano /etc/crypttab /etc/fstab

: <<'END'
    ubuntu@ip-172-31-15-182:~$ cat /etc/crypttab 
    # <target name> <source device>         <key file>      <options>
    md0_crypt UUID=627322a8-9c36-4aa4-ba0f-bba122252f8f     none    luks,noauto
    lv1_crypt UUID=d441aa85-38e4-414e-bf93-2d6334997b7e     none    luks,noauto

    ubuntu@ip-172-31-15-182:~$ cat /etc/fstab 
    LABEL=cloudimg-rootfs   /        ext4   discard,errors=remount-ro       0 1
    LABEL=UEFI      /boot/efi       vfat    umask=0077      0 1
    UUID=cbf994f5-673a-493a-b254-7cf6bee075c5 /mnt/raid6NoLVM ext4 defaults,noauto 0 0
    UUID=721b459c-921c-462e-8835-132e64c28921 /mnt/raid6LVM xfs defaults,noauto 0 0
END

sudo mdadm --verbose --examine --scan | sudo tee -a /etc/mdadm/mdadm.conf

sudo nano /etc/systemd/system/crypto-mount-lvm.service

: <<'END'
    ubuntu@ip-172-31-15-182:~$ cat /etc/systemd/system/crypto-mount-lvm.service
    [Unit]
    Description=Open-Mount and Unmount-Close LVM LUKS RAID6 Volume
    After=local-fs-pre.targe

    [Service]
    Type=oneshot
    RemainAfterExit=yes
    ExecStartPre=-/bin/mkdir -p /mnt/raid6LVM
    ExecStartPre=-/bin/umount /mnt/raid6LVM
    ExecStart=/usr/lib/systemd/systemd-cryptsetup attach lv1_crypt /dev/vg1/lv1
    ExecStart=/bin/mount /dev/mapper/lv1_crypt /mnt/raid6LVM
    ExecStop=/bin/umount /mnt/raid6LVM
    ExecStopPost=-/usr/lib/systemd/systemd-cryptsetup detach lv1_crypt

    # Omitting [Install] section to keep the service manual-start only
END

sudo nano /etc/systemd/system/crypto-mount.service

: <<'END'
    ubuntu@ip-172-31-15-182:~$ cat /etc/systemd/system/crypto-mount.service
    [Unit]
    Description=Open-Mount and Unmount-Close LUKS RAID6 Volume
    After=local-fs-pre.targe

    [Service]
    Type=oneshot
    RemainAfterExit=yes
    ExecStartPre=-/bin/mkdir -p /mnt/raid6NoLVM
    ExecStartPre=-/bin/umount /mnt/raid6NoLVM
    ExecStart=/usr/lib/systemd/systemd-cryptsetup attach md0_crypt /dev/md0
    ExecStart=/bin/mount /dev/mapper/md0_crypt /mnt/raid6NoLVM
    ExecStop=/bin/umount /mnt/raid6NoLVM
    ExecStopPost=-/usr/lib/systemd/systemd-cryptsetup detach md0_crypt

    # Omitting [Install] section to keep the service manual-start only
END

sudo update-initramfs -u

sudo reboot

df -h && sudo systemctl start crypto-mount-lvm.service && df -h

sudo systemctl start crypto-mount.service && df -h
