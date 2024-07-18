ref: https://github.com/jaime-10-reis/Project-AWS/blob/main/Debian/RAID%205%20%26%206

✵ RAID 5 & 6 Red Hat-Debian


➜ lsblk
➜ gdisk /dev/xvdf
ℹ️ do with the following sequence:
✔️ o -> y
✔️ n -> ENTER -> ENTER -> ENTER -> fd00
✔️ w -> y
➜ gdisk /dev/xvdg
ℹ️ do with the following sequence:
✔️ o -> y
✔️ n -> ENTER -> ENTER -> ENTER -> fd00
✔️ w -> y
➜ gdisk /dev/xvdh
ℹ️ do with the following sequence:
✔️ o -> y
✔️ n -> ENTER -> ENTER -> ENTER -> fd00
✔️ w -> y
➜ mdadm --create /dev/md0 --level 5 --raid-devices 3 /dev/xvdf1 /dev/xvdg1 /dev/xvdh1
➜ mdadm --detail /dev/md0
ℹ️ wait until "resync" appears
➜ cryptsetup luksFormat --hash=sha512 --key-size=512 --cipher=aes-xts-plain64 --verify-passphrase /dev/md0
#### YES -> (your password)
➜ cryptsetup luksOpen /dev/md0 md0_crypt
####
(your password)


♤ to add a key:
➜ cryptsetup luksAddKey /dev/md0
####
existing password -> new password
➜ cryptsetup luksDump -v /dev/md0
ℹ️ to see the keys you have


➜ pvcreate /dev/mapper/md0_crypt
➜ vgcreate vg0 /dev/mapper/md0_crypt
➜ lvcreate -n vg0lv0 -l +75%FREE vg0
➜ lvcreate -n vg0lv1 -l +100%FREE vg0
➜ lvdisplay

➜ mkfs.xfs /dev/vg0/vg0lv0
ℹ️ you can use "xfs" or "ext4"
➜ mkfs.xfs /dev/vg0/vg0lv1
ℹ️ you can use "xfs" or "ext4"
➜ df -hT

➜ cd /mnt/
➜ mkdir raid5_1
➜ mkdir raid5_2
➜ mount /dev/vg0/vg0lv0 /mnt/raid5_1
➜ mount /dev/vg0/vg0lv1 /mnt/raid5_2

➜ cat /etc/mtab
ℹ️ copy the two lines where you have the vg
➜ nano /etc/fstab/
ℹ️ paste here and acrecentar "nofail,"

Reboot? Next:
➜ cryptsetup luksOpen /dev/md0 md0_crypt
ℹ️ one of the word-passes you defined
➜ mount -a


✵ RAID 5 & 6 Red Hat-Debian


➜ lsblk
➜ gdisk /dev/xvdf
ℹ️ do with the following sequence:
✔️ o -> y
✔️ n -> ENTER -> ENTER -> ENTER -> fd00
✔️ w -> y
➜ gdisk /dev/xvdg
ℹ️ do with the following sequence:
✔️ o -> y
✔️ n -> ENTER -> ENTER -> ENTER -> fd00
✔️ w -> y
➜ gdisk /dev/xvdh
ℹ️ do with the following sequence:
✔️ o -> y
✔️ n -> ENTER -> ENTER -> ENTER -> fd00
✔️ w -> y
➜ mdadm --create /dev/md0 --level 6 --raid-devices 4 /dev/xvdf1 /dev/xvdg1 /dev/xvdh1 /dev/xvdi1
➜ mdadm --detail /dev/md0
ℹ️ wait until "resync" appears
➜ pvcreate /dev/md0
➜ vgcreate vg1 /dev/md0
➜ lvcreate -n vg1lv0 -l +50%FREE vg1
➜ lvcreate -n vg1lv1 -l +100%FREE vg1
➜ lvdisplay

➜ cryptsetup luksFormat --hash=sha512 --key-size=512 --cipher=aes-xts-plain64 --verify-passphrase /dev/vg1/vg1lv0
ℹ️ YES -> (new password)
➜ cryptsetup luksFormat --hash=sha512 --key-size=512 --cipher=aes-xts-plain64 --verify-passphrase /dev/vg1/vg1lv1
ℹ️ YES -> (another new password)
➜ cryptsetup luksOpen /dev/vg1/vg1lv0 vg1lv0_crypt
ℹ️ (the password you used in this volume)
➜ cryptsetup luksOpen /dev/vg1/vg1lv1 vg1lv1_crypt
ℹ️ (the password you used in this volume)
➜ mkfs.xfs /dev/mapper/vg1lv0_crypt
ℹ️ you can use "xfs" or "ext4"
➜ mkfs.xfs /dev/mapper/vg1lv1_crypt
ℹ️ you can use "xfs" or "ext4"

➜ cd /mnt/
➜ mkdir raid6_1
➜ mkdir raid6_2
➜ mount /dev/mapper/vg1lv0_crypt /mnt/raid6_1/
➜ mount /dev/mapper/vg1lv1_crypt /mnt/raid6_2/

➜ cat /etc/mtab
ℹ️ copy the two lines where you have the vg
➜ nano /etc/fstab/
ℹ️ paste here and acrecentar ",nofail,"

Reboot? Next:
➜ cryptsetup luksOpen /dev/vg1/vg1lv0 vg1lv0_crypt
ℹ️ one of the word-passes you defined
➜ cryptsetup luksOpen /dev/vg1/vg1lv1 vg1lv1_crypt
ℹ️ one of the word-passes you defined
➜ mount -a

DONE
