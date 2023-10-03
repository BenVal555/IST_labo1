# IST_labo1

## TASK 1: EXPLORE BLOCK DEVICES AND FILESYSTEMS

1. Using the lsblk command, list the existing block devices
```shell
ubuntu@primary:/$ lsblk
NAME    MAJ:MIN RM   SIZE RO TYPE MOUNTPOINTS
loop0     7:0    0  59.3M  1 loop /snap/core20/2019
loop1     7:1    0  35.5M  1 loop /snap/snapd/20102
loop2     7:2    0 109.6M  1 loop /snap/lxd/24326
loop3     7:3    0     4K  1 loop /snap/bare/5
loop4     7:4    0   868K  1 loop /snap/multipass-sshfs/147
sda       8:0    0    50G  0 disk 
├─sda1    8:1    0  49.9G  0 part /
└─sda15   8:15   0    99M  0 part /boot/efi
vda     252:0    0    52K  1 disk
```

On which block device is the boot partition mounted? In the /dev directory find the special file corresponding to this
block device. With ls -l list its metadata.

```shell
ubuntu@primary:/$ df /boot/efi/
Filesystem     1K-blocks  Used Available Use% Mounted on
/dev/sda15         99800  6418     93383   7% /boot/efi
```

```shell
ubuntu@primary:/$ sudo ls -l /boot/efi/
total 1
drwx------ 4 root root 512 Sep 14 04:19 EFI
```

On which block device is the root (/) partition mounted? What is the name of its special file?

```shell
ubuntu@primary:/$ df /
Filesystem     1K-blocks    Used Available Use% Mounted on
/dev/sda1       50622456 1664508  48941564   4% /
```

```shell
ubuntu@primary:/$ sudo ls -l /root/
total 4
drwx------ 3 root root 4096 Sep 27 10:50 snap
```

With hdparm -t do a timing test on the boot partition. What throughput do you get?

```shell
ubuntu@primary:/$ sudo hdparm -t /dev/sda15 

/dev/sda15:
 Timing buffered disk reads:  98 MB in  0.07 seconds = 1378.55 MB/sec
```

## TASK 1 REDO BY BEN
1. Using the lsblk command, list the existing block devices

```shell
ben@ben-virtual-machine:~/Desktop$ lsblk
NAME   MAJ:MIN RM   SIZE RO TYPE MOUNTPOINTS
fd0      2:0    1     4K  0 disk 
loop0    7:0    0  63.4M  1 loop /snap/core20/1974
loop1    7:1    0 349.7M  1 loop /snap/gnome-3-38-2004/143
loop2    7:2    0 237.2M  1 loop /snap/firefox/2987
loop3    7:3    0     4K  1 loop /snap/bare/5
loop4    7:4    0  73.9M  1 loop /snap/core22/858
loop5    7:5    0 485.5M  1 loop /snap/gnome-42-2204/120
loop6    7:6    0  12.3M  1 loop /snap/snap-store/959
loop7    7:7    0  91.7M  1 loop /snap/gtk-common-themes/1535
loop8    7:8    0  53.3M  1 loop /snap/snapd/19457
loop9    7:9    0   452K  1 loop /snap/snapd-desktop-integration/83
sda      8:0    0    20G  0 disk 
├─sda1   8:1    0     1M  0 part 
├─sda2   8:2    0   513M  0 part /boot/efi
└─sda3   8:3    0  19.5G  0 part /var/snap/firefox/common/host-hunspell
                                 /
sr0     11:0    1 155.3M  0 rom  /media/ben/CDROM
sr1     11:1    1  1024M  0 rom  
```
On which block device is the boot partition mounted? In the /dev directory find the special file corresponding to this
block device. With ls -l list its metadata.

It's mounted on sda2.
```shell
ben@ben-virtual-machine:/dev$ ls -l sda2
brw-rw---- 1 root disk 8, 2 Okt  1 11:29 sda2
```
With hdparm -t do a timing test on the boot partition. What throughput do you get?
```shell
ben@ben-virtual-machine:/dev$ sudo hdparm -t /dev/sda2
[sudo] password for ben: 

/dev/sda2:
 Timing buffered disk reads: 512 MB in  1.45 seconds = 353.11 MB/sec
```

2. Convince yourself that the special file representing the root (/) partition can be read just like any other file. As it contains binary data, just opening it with less will mess up the terminal, so use the xxd hexdump utility.

To see how xxd works, create a small text file and open it with xxd -a.
```shell
ben@ben-virtual-machine:~/Desktop$ echo "Hello, World!" > testfile.txt
ben@ben-virtual-machine:~/Desktop$ xxd -a testfile.txt
00000000: 4865 6c6c 6f2c 2057 6f72 6c64 210a       Hello, World!.
```

Now open the special file with the same command. You may pipe its output into less. What do you see? If your root partition uses LVM (verify with lsblk), you should see text strings containing volume group configuration information.
I see a hexadecimal representation of data on the boot partition.

3. As the special file represents all the blocks of a partition, the content of all files of the root partition should be there. Pick a text file at random (for example a file in /usr/share/doc/) and try to find its content in the special file.


## TASK 2: PREPARE AND PARTITION A DISK

Before you plug in the disk, list the existing block devices. Using the findmnt command find all the partitions that are already mounted.

### Listing block devices
```shell
en@ben-virtual-machine:~/Desktop$ lsblk
NAME   MAJ:MIN RM   SIZE RO TYPE MOUNTPOINTS
fd0      2:0    1     4K  0 disk 
loop0    7:0    0     4K  1 loop /snap/bare/5
loop1    7:1    0  63.4M  1 loop /snap/core20/1974
loop2    7:2    0  63.5M  1 loop /snap/core20/2015
loop3    7:3    0  73.9M  1 loop /snap/core22/858
loop4    7:4    0  73.9M  1 loop /snap/core22/864
loop5    7:5    0 237.2M  1 loop /snap/firefox/2987
loop6    7:6    0 485.5M  1 loop /snap/gnome-42-2204/120
loop7    7:7    0 349.7M  1 loop /snap/gnome-3-38-2004/143
loop8    7:8    0   497M  1 loop /snap/gnome-42-2204/141
loop9    7:9    0  91.7M  1 loop /snap/gtk-common-themes/1535
loop10   7:10   0  12.3M  1 loop /snap/snap-store/959
loop11   7:11   0  53.3M  1 loop /snap/snapd/19457
loop12   7:12   0  40.8M  1 loop /snap/snapd/20092
loop13   7:13   0   452K  1 loop /snap/snapd-desktop-integration/83
sda      8:0    0    20G  0 disk 
├─sda1   8:1    0     1M  0 part 
├─sda2   8:2    0   513M  0 part /boot/efi
└─sda3   8:3    0  19.5G  0 part /var/snap/firefox/common/host-hunspell
                                 /
sr0     11:0    1 155.3M  0 rom  /media/ben/CDROM
sr1     11:1    1   4.7G  0 rom  /media/ben/Ubuntu 22.04.3 LTS amd64
```
### Listing partitions
```shell
ben@ben-virtual-machine:/usr/share/doc/zip$ findmnt --real
TARGET                                   SOURCE     FSTYPE     OPTIONS
/                                        /dev/sda3  ext4       rw,relatime,errors=remount-ro
├─/run/user/1000/doc                     portal     fuse.porta rw,nosuid,nodev,relatime,user_id=1000,group_id=1000
├─/snap/core20/1974                      /dev/loop0 squashfs   ro,nodev,relatime,errors=continue,threads=single
├─/snap/bare/5                           /dev/loop3 squashfs   ro,nodev,relatime,errors=continue,threads=single
├─/snap/firefox/2987                     /dev/loop2 squashfs   ro,nodev,relatime,errors=continue,threads=single
├─/snap/core22/858                       /dev/loop4 squashfs   ro,nodev,relatime,errors=continue,threads=single
├─/snap/gnome-42-2204/120                /dev/loop5 squashfs   ro,nodev,relatime,errors=continue,threads=single
├─/snap/gnome-3-38-2004/143              /dev/loop1 squashfs   ro,nodev,relatime,errors=continue,threads=single
├─/snap/snap-store/959                   /dev/loop6 squashfs   ro,nodev,relatime,errors=continue,threads=single
├─/snap/gtk-common-themes/1535           /dev/loop7 squashfs   ro,nodev,relatime,errors=continue,threads=single
├─/snap/snapd/19457                      /dev/loop8 squashfs   ro,nodev,relatime,errors=continue,threads=single
├─/snap/snapd-desktop-integration/83     /dev/loop9 squashfs   ro,nodev,relatime,errors=continue,threads=single
├─/boot/efi                              /dev/sda2  vfat       rw,relatime,fmask=0077,dmask=0077,codepage=437,iochars
├─/media/ben/CDROM                       /dev/sr0   iso9660    ro,nosuid,nodev,relatime,nojoliet,check=s,map=n,blocks
└─/var/snap/firefox/common/host-hunspell /dev/sda3[/usr/share/hunspell]
                                                    ext4       ro,noexec,noatime,errors=remount-ro
```

2. List again the block devices. Which new block devices and special files appeared? These represent the disk and its partitions you just attached.

**Answer:** As we can see below, the new block device sdb appeared in the list. Since the disk has just been created it doesn't have any partitions yet.
```shell
ben@ben-virtual-machine:~/Desktop$ lsblk
NAME   MAJ:MIN RM   SIZE RO TYPE MOUNTPOINTS
fd0      2:0    1     4K  0 disk 
loop0    7:0    0  63.4M  1 loop /snap/core20/1974
loop1    7:1    0     4K  1 loop /snap/bare/5
loop2    7:2    0  73.9M  1 loop /snap/core22/858
loop3    7:3    0  63.5M  1 loop /snap/core20/2015
loop4    7:4    0  73.9M  1 loop /snap/core22/864
loop5    7:5    0 237.2M  1 loop /snap/firefox/2987
loop6    7:6    0 349.7M  1 loop /snap/gnome-3-38-2004/143
loop7    7:7    0 485.5M  1 loop /snap/gnome-42-2204/120
loop8    7:8    0  12.3M  1 loop /snap/snap-store/959
loop9    7:9    0  40.8M  1 loop /snap/snapd/20092
loop10   7:10   0  91.7M  1 loop /snap/gtk-common-themes/1535
loop11   7:11   0   452K  1 loop /snap/snapd-desktop-integration/83
loop12   7:12   0  53.3M  1 loop /snap/snapd/19457
loop13   7:13   0   497M  1 loop /snap/gnome-42-2204/141
sda      8:0    0    20G  0 disk 
├─sda1   8:1    0     1M  0 part 
├─sda2   8:2    0   513M  0 part /boot/efi
└─sda3   8:3    0  19.5G  0 part /var/snap/firefox/common/host-hunspell
                                 /
sdb      8:16   0     1G  0 disk 
sr0     11:0    1 155.3M  0 rom  /media/ben/CDROM
sr1     11:1    1   4.7G  0 rom  /media/ben/Ubuntu 22.04.3 LTS amd64
```

3. Create a partition table on the disk and create two partitions of equal size using the parted tool [...]
```shell
ben@ben-virtual-machine:~/Desktop$ sudo parted /dev/sdb
GNU Parted 3.4
Using /dev/sdb
Welcome to GNU Parted! Type 'help' to view a list of commands.
(parted) print                                                            
Error: /dev/sdb: unrecognised disk label
Model: VMware, VMware Virtual S (scsi)                                    
Disk /dev/sdb: 1074MB
Sector size (logical/physical): 512B/512B
Partition Table: unknown
Disk Flags: 
(parted) mktable msdos
(parted) print free                                                       
Model: VMware, VMware Virtual S (scsi)
Disk /dev/sdb: 1074MB
Sector size (logical/physical): 512B/512B
Partition Table: msdos
Disk Flags: 

Number  Start  End     Size    Type  File system  Flags
        1024B  1074MB  1074MB        Free Space

(parted) mkpart primary fat32 0 537MB                                      
Warning: The resulting partition is not properly aligned for best performance:
1s % 2048s != 0s
Ignore/Cancel? Ignore                                                     
(parted) print
Model: VMware, VMware Virtual S (scsi)
Disk /dev/sdb: 1074MB
Sector size (logical/physical): 512B/512B
Partition Table: msdos
Disk Flags: 

Number  Start  End    Size   Type     File system  Flags
 1      512B   537MB  537MB  primary  fat32        lba

(parted) mkpart primary ext4 537MB 1074MB
Warning: The resulting partition is not properly aligned for best performance:
1048829s % 2048s != 0s
Ignore/Cancel? Ignore                                                     
(parted) print                                                            
Model: VMware, VMware Virtual S (scsi)
Disk /dev/sdb: 1074MB
Sector size (logical/physical): 512B/512B
Partition Table: msdos
Disk Flags: 

Number  Start  End     Size   Type     File system  Flags
 1      512B   537MB   537MB  primary  fat32        lba
 2      537MB  1074MB  537MB  primary  ext4         lba

(parted) quit                                                             
Information: You may need to update /etc/fstab.

ben@ben-virtual-machine:~/Desktop$ ls /dev/sdb*                           
/dev/sdb  /dev/sdb1  /dev/sdb2
```
4. Format the two partitions using the mkfs command.
5. Create two empty directories in the /mnt directory as mount points, called part1 and part2. Mount the newly created file systems in these directories.
6. How much free space is available on these filesystems? Use the df command to find out. What does the -h option do?
**Answer:** /dev/sdb1 has 512M available and /dev/sdb2 has 464M available. The -h option is used to make the ouput more human friendly by displaying sizes in easy to read formats.
```shell
ben@ben-virtual-machine:~/Desktop$ sudo mkfs.vfat /dev/sdb1
[sudo] password for ben: 
mkfs.fat 4.2 (2021-01-31)
ben@ben-virtual-machine:~/Desktop$ sudo mkfs.ext4 /dev/sdb2
mke2fs 1.46.5 (30-Dec-2021)
Creating filesystem with 131040 4k blocks and 131072 inodes
Filesystem UUID: 53580d06-65f6-4853-8a18-2bf64f74a0a0
Superblock backups stored on blocks: 
	32768, 98304

Allocating group tables: done                            
Writing inode tables: done                            
Creating journal (4096 blocks): done
Writing superblocks and filesystem accounting information: done

ben@ben-virtual-machine:~/Desktop$ sudo mkdir /mnt/part1
ben@ben-virtual-machine:~/Desktop$ sudo mkdir /mnt/part2
ben@ben-virtual-machine:~/Desktop$ sudo mount /dev/sdb1 /mnt/part1
ben@ben-virtual-machine:~/Desktop$ sudo mount /dev/sdb2 /mnt/part2
ben@ben-virtual-machine:~/Desktop$ df -h /mnt/part1 /mnt/part2
Filesystem      Size  Used Avail Use% Mounted on
/dev/sdb1       512M  4.0K  512M   1% /mnt/part1
/dev/sdb2       464M   24K  428M   1% /mnt/part2
```
