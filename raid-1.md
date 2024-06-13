# Setting Up Software RAID 1 on Linux

This guide will walk you through setting up a Software RAID 1 (mirroring) on Linux using `mdadm`.

## Step 1: List the Available Disks

Open a terminal and list the available disks using one of the following commands:

```sh
sudo fdisk -l
# or
lsblk
```

Lets assume disk1 and disk2 are their names.

## Step 2: Create a New MBR Partition Table on Both Disks

Replace <disk1> and <disk2> with your actual disk identifiers (e.g., sdb, sdc).

```sh
sudo parted /dev/<disk1> mklabel msdos
sudo parted /dev/<disk2> mklabel msdos
```

## Step 3: Create a New Partition on Each Disk

Replace <disk1> with your actual disk identifier (e.g., sdb).

```sh
sudo fdisk /dev/<disk1>
```

Follow these instructions:

  1. Type n to create a new partition.
  2. Type p to select a primary partition.
  3. Type 1 to create /dev/<disk1>1.
  4. Press Enter to choose the default first sector.
  5. Press Enter to choose the default last sector. This partition will span across the entire drive.
  6. Type p to print information about the newly created partition.
  7. Type t to change the partition type.
  8. Enter fd to set the partition type to Linux raid autodetect.
  9. Type p again to check the partition type.
  10.Type w to write the changes to the disk.

Repeat these steps for <disk2>.


## Step 4: Install mdadm

```sh
sudo apt install mdadm
```

## Step 5: Examine the Partitions

Check if the partitions are set to type fd (Linux raid autodetect).

```sh
sudo mdadm --examine /dev/<disk1> /dev/<disk2>
```


## Step 6: Create the RAID 1 Array

Replace <disk1n1> and <disk2n1> with your actual partition identifiers (e.g., sdb1, sdc1).

```sh
sudo mdadm --create /dev/md0 --level=mirror --raid-devices=2 /dev/<disk1n1> /dev/<disk2n1>
```

## Step 7: Verify the RAID Array

Check if the RAID array is active:

```sh
cat /proc/mdstat
```

Get detailed information about the RAID array:

```sh
sudo mdadm --detail /dev/md0
```

Examine the RAID partitions:

```sh
sudo mdadm --examine /dev/<disk1n1> /dev/<disk2n1>
```

## Step 8: Format the RAID Array

Format the RAID array to ext4:

```sh
sudo mkfs.ext4 /dev/md0
```

## Step 9: Mount the RAID Volume

Create a mount point and mount the RAID volume:

```sh
sudo mkdir /mnt/raid1
sudo mount /dev/md0 /mnt/raid1
```

## Step 10: Auto Mount on Startup

Edit /etc/fstab to mount the RAID array automatically on startup. Add the following line:

```sh
/dev/md0  /mnt/raid1   ext4    defaults,x-gvfs-show   0   0
```

The x-gvfs-show option will let you see your RAID1 in the sidebar of your file manager.
