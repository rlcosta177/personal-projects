## Files in HDD cant be deleted

1) find the index of the drive:
    - lsblk
  
2) define the possible problem:
    - e.g /dev/sda2 on /media/user/68948A62948A331C type fuseblk (ro,nosuid,nodev,relatime,user_id=0,group_id=0,default_permissions,allow_other,blksize=4096,uhelper=udisks2)
    - the file type is 'fuseblk' which is ro(read-only)
  
3) following that, change the file type to a rw(read-write):
    - sudo mount -o remount,rw /media/user/68948A62948A331C
  
4) Other possible problems & fixes:
    - Ownership and Permissions -> ls -l /media/user/68948A62948A331C
    - Filesystem Errors -> use fsck to check for errors(unmount the partition first to prevent data corruption)
