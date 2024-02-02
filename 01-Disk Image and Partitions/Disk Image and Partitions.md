Disk Image and Partitions

Integrity Check

A disk image has been collected of the suspect's device. MD5 and SHA1 hashes have been created prior to collection to verify the collection has completed properly.
The disk image MD5 and SHA1 hashes were checked against the reference list through the command line.

md5deep Disk_Image_ID-20210327.001 -bewM hash_reference.txt

sha1deep Disk_Image_ID-20210327.001 -bewM hash_reference.txt

Identifying OS Name, Accounts and Partitions

It is important to identify the OS Name/Version, the partitions that exist on the system and accounts to be able to frame your investigation. Knowing the OS Name and version will allow you to know where to look for certain artifacts as Windows systems will differ from Linux systems. Getting a run down of partitions will allow you to see the allocated vs unallocated space to determine if there is anything missing or deleted and needs to be recovered further.

Fdisk command allows us to view the partition table of the device.

fdisk -l Disk_Image_ID-20210327.001

Fsstat command will give file system information of a partition. It requires the offset of the partition otherwise it will not be able to run.

fsstat -o 2048 Disk_Image_ID-20210327.001
fsstat -o 104448 Disk_Image_ID-20210327.001
fsstat -o 61890560 Disk_Image_ID-20210327.001

The output from these commands shows that this system is running on Windows XP OS with a NTFS file system type. The first partition includes files the OS needs to boot. The second partition contains user files, which is what will be mounted and investigated. The third partition is a recovery partition allowing a user to factory reset their system.

Alternate commands

mmls command shows the partition table but rounds the sizes down.

mmls -B Disk_Image_ID-20210327.001

gparted command an an option that shows the partitions in a GUI.

gparted Disk_Image_ID-20210327.001

Next we will look into analyzing the registry files for further analysis.