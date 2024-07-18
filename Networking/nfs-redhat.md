ref: https://github.com/VitorCarreiro/NFS

# NFS for Redhat

NFS Config

**Install NFS**

`yum install nfs-utils`

**Start and enable NFS**

`systemctl start nfs-server.service` and `systemctl enable nfs-server.service`

**Create folder that you'd like to export to clients**

`mkdir -p /mnt/nfs_test`

**Configure the /etc/exports file to be something like this**

`/mnt/nfs_test *(rw,sync,no_subtree_check,no_root_squash)`

**Use this command to export or unexport all directories**

`exportfs -a`

**Add a user to the nfs folder**

`adduser ann --home /mnt/nfs_test/ann`



➡️ **Now set up NFS client** ⬅️ 🔖 

**Install NFS**

`yum install nfs-utils`

**Create the folder you made on server**

`mkdir /mnt/nfs_test`

**Mount the system**

`mount -t nfs (Server.IP):/mnt/nfs_test/ /mnt/nfs_test/`

**Do `cat /etc/mtab` and copy the last line**

**Now go to `nano /etc/fstab` and paste the line there with ,nofail, between some words**

And that is it, you should now be able to enter the folder and create files!



