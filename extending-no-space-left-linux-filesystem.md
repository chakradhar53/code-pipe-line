Example
Use the df -h command to verify that the root partition mounted under "/" is full (100%). In the following example, /dev/nvme0n1p1 is using 100% of its space.

[ec2-user ~]$ df -h
Filesystem      Size  Used Avail Use% Mounted on
devtmpfs        460M     0  475M   0% /dev
tmpfs           478M     0  492M   0% /dev/shm
tmpfs           478M  432K  492M   1% /run
tmpfs           478M     0  492M   0% /sys/fs/cgroup
/dev/nvme0n1p1  8.0G  8.0G  664K 100% /
tmpfs            96M     0   99M   0% /run/user/1000
If you are running on aws cloud, you can extend the root EBS Volume in AWS Console (ec2 console) or CLI, read here https://aws.amazon.com/premiumsupport/knowledge-center/expand-ebs-root-volume-windows/

The following example shows that the root EBS volume block device (/dev/nvme0n1) is 9 GiB, and the root partition (partition 1) is already 8 GiB (partition is 100% used).

$ lsblk
NAME          MAJ:MIN    RM SIZE RO TYPE    MOUNTPOINT
nvme0n1       259:0      0   9G  0  disk
├─nvme0n1p1   259:1      0   8G  0  part   /
└─nvme0n1p128 259:2      0   1M  0  part
Problem
If you attempt to increase the root partition (partition 1), you receive one of the following errors:

$ sudo growpart /dev/nvme0n1 1
/bin/growpart: line 248: /tmp/growpart.fklt5u/dump.out: No space left on device
FAILED: failed to dump sfdisk info for /dev/nvme0n1
-or-

$ sudo growpart /dev/nvme0n1 1
CHANGED: partition=1 start=4096 old: size=16773087 end=16777183 new: size=18870239 end=18874335
FAILED: failed: sfdisk --list /dev/nvme0n1
Resolution
To avoid a No space left on the block device error, mount the temporary file system tmpfs to the /tmp mount point. This creates a 10 M tmpfs mounted to /tmp.

$ sudo mount -o size=10M,rw,nodev,nosuid -t tmpfs tmpfs /tmp
then try again run the growpart command to grow the size of the root partition or partition 1. Replace /dev/nvme0n1 with your root partition.

$ sudo growpart /dev/nvme0n1 1 
CHANGED: partition=1 start=4096 old: size=16773087 end=16777183 new: size=18870239 end=18874335
Run the lsblk command to verify that partition 1 is expanded to 9 GiB.

$ lsblk
NAME          MAJ:MIN RM SIZE RO TYPE MOUNTPOINT
nvme0n1       259:0    0   9G  0 disk
├─nvme0n1p1   259:1    0   9G  0 part /
└─nvme0n1p128 259:2    0   1M  0 part
then, run resize2fs to expand the file system

sudo resize2fs /dev/nvme0n1p1
After expanding the file system, use the df -h command to verify that the OS can see the additional space (partition is 89% used).

$ df -h
Filesystem      Size  Used Avail Use% Mounted on
devtmpfs        960M     0  960M   0% /dev
tmpfs           978M     0  978M   0% /dev/shm
tmpfs           978M  392K  978M   1% /run
tmpfs           978M     0  978M   0% /sys/fs/cgroup
/dev/nvme0n1p1  9.0G  8.0G 1022M  89% /
tmpfs           196M     0  196M   0% /run/user/1000
tmpfs            10M     0   10M   0% /tmp
Run the unmount command to unmount the tmpfs file system.

$ sudo umount /tmp
source : https://aws.amazon.com/premiumsupport/knowledge-center/ebs-volume-size-increase/
