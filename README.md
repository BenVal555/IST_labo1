# IST_labo1

## TASK 1 : EXPLORE BLOCK DEVICES AND FILESYSTEMS
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

2. Convince yourself that the special file representing the root (/) partition can be read just like any other file. As
it contains binary data, just opening it with less will mess up the terminal, so use the xxd hexdump utility.

To see how xxd works, create a small text file and open it with xxd -a.
```shell
ben@ben-virtual-machine:~/Desktop$ echo "Hello, World!" > testfile.txt
ben@ben-virtual-machine:~/Desktop$ xxd -a testfile.txt
00000000: 4865 6c6c 6f2c 2057 6f72 6c64 210a       Hello, World!.
```

Now open the special file with the same command. You may pipe its output into less. What do you see? If your root
partition uses LVM (verify with lsblk), you should see text strings containing volume group configuration information.

*We can see a hexadecimal representation of data on the boot partition.*

3. As the special file represents all the blocks of a partition, the content of all files of the root partition should
be there. Pick a text file at random (for example a file in /usr/share/doc/) and try to find its content in the special
file.


## TASK 2: PREPARE AND PARTITION A DISK

Before you plug in the disk, list the existing block devices. Using the findmnt command find all the partitions that
are already mounted.

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

2. List again the block devices. Which new block devices and special files appeared? These represent the disk and its
partitions you just attached.

*As we can see below, the new block device sdb appeared in the list. Since the disk has just been created it doesn't
have any partitions yet.*
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
5. Create two empty directories in the /mnt directory as mount points, called part1 and part2. Mount the newly created
file systems in these directories.
6. How much free space is available on these filesystems? Use the df command to find out. What does the -h option do?

*/dev/sdb1 has 512M available and /dev/sdb2 has 464M available. The -h option is used to make the ouput more human
friendly by displaying sizes in easy to read formats.*
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
## TASK 3: EXPLORE THE FILE SYSTEM SUPPORT IN THE KERNEL
Find out which file systems the kernel supports right now. The kernel makes information about itself available to
userspace programs in a pseudo file system that is mounted at /proc. The files in that file system describe kernel
objects.

List the content of /proc. What is the version of the kernel in /proc/version?

*The version is 6.2.0-33-generic (Linux).*
```shell
ben@ben-virtual-machine:~/Desktop$ cat /proc/version
Linux version 6.2.0-33-generic (buildd@lcy02-amd64-073) (x86_64-linux-gnu-gcc-11 (Ubuntu 11.4.0-1ubuntu1~22.04) 11.4.0, GNU ld (GNU Binutils for Ubuntu) 2.38) #33~22.04.1-Ubuntu SMP PREEMPT_DYNAMIC Thu Sep  7 10:33:52 UTC 2
```
The directories with numbers represent the running processes. The numbers are the process ids. Display the process id
of your bash session with echo $$. List the information in the corresponding directory. What was the command line that
started this process (look in cmdline)?

Below is the process id of my bash session.
```shell
ben@ben-virtual-machine:~/Desktop$ echo $$
2302
```

Command line that started this process: bash, as seen below
```shell
ben@ben-virtual-machine:~/Desktop$ cat /proc/2302/cmdline
bash
```

The kernel lists the file systems it supports right now file filesystems. List them.
```shell
ben@ben-virtual-machine:~/Desktop$ cat /proc/filesystems 
nodev	sysfs
nodev	tmpfs
nodev	bdev
nodev	proc
nodev	cgroup
nodev	cgroup2
nodev	cpuset
nodev	devtmpfs
nodev	configfs
nodev	debugfs
nodev	tracefs
nodev	securityfs
nodev	sockfs
nodev	bpf
nodev	pipefs
nodev	ramfs
nodev	hugetlbfs
nodev	devpts
	ext3
	ext2
	ext4
	squashfs
	vfat
nodev	ecryptfs
	fuseblk
nodev	fuse
nodev	fusectl
nodev	mqueue
nodev	pstore
nodev	autofs
nodev	binfmt_misc
	iso9660
```

Can you find the proc filesystem itself in the list? How is it tagged? All file systems with that tag are pseudo file
systems.

*Yes. It is tagged "nodev".*

List the real (non-pseudo) file systems.
```shell
ben@ben-virtual-machine:~/Desktop$ cat /proc/filesystems | grep -v "nodev" | awk '{print $NF}'
ext3
ext2
ext4
squashfs
vfat
fuseblk
iso9660
```

Find out which file systems the kernel is able to support by looking at the available kernel modules. The files
containing kernel modules can be found at lib/modules/<kernel version>/kernel/fs. List them.
```shell
ben@ben-virtual-machine:~/Desktop$ ls /lib/modules/6.2.0-33-generic/kernel/fs
9p      bfs             coda    f2fs      hfs      ksmbd       nfsd    omfs       quota         ubifs
adfs    binfmt_misc.ko  cramfs  fat       hfsplus  lockd       nilfs2  orangefs   reiserfs      udf
affs    btrfs           dlm     freevxfs  hpfs     minix       nls     overlayfs  romfs         ufs
afs     cachefiles      efs     fscache   isofs    netfs       ntfs    pstore     shiftfs.ko    vboxsf
autofs  ceph            erofs   fuse      jffs2    nfs         ntfs3   qnx4       smbfs_common  xfs
befs    cifs            exfat   gfs2      jfs      nfs_common  ocfs2   qnx6       sysv          zonefs
```

When a new disk is inserted the kernel knows which file system to activate by looking at a label that indicates the
type of file system. That label is part of the partition metadata (called signature). Use the blkid command to list the
metadata of all known partitions (mounted or not). Note that you might need to run the command with admin permissions
to display all partitions metadata.
```shell
ben@ben-virtual-machine:~/Desktop$ sudo blkid
[sudo] password for ben: 
/dev/sda3: UUID="b2a1a2f6-0070-45c6-ba4c-29595b38ffad" BLOCK_SIZE="4096" TYPE="ext4" PARTUUID="ecc9bdf3-1c00-4e20-a0d2-a3f1bfee3ae7"
/dev/loop1: TYPE="squashfs"
/dev/loop8: TYPE="squashfs"
/dev/sdb2: UUID="53580d06-65f6-4853-8a18-2bf64f74a0a0" BLOCK_SIZE="4096" TYPE="ext4" PARTUUID="886d0b38-02"
/dev/sdb1: UUID="2EF0-D8A9" BLOCK_SIZE="512" TYPE="vfat" PARTUUID="886d0b38-01"
/dev/loop6: TYPE="squashfs"
/dev/loop13: TYPE="squashfs"
/dev/loop4: TYPE="squashfs"
/dev/loop11: TYPE="squashfs"
/dev/sr0: BLOCK_SIZE="2048" UUID="2023-10-01-11-02-18-00" LABEL="CDROM" TYPE="iso9660"
/dev/loop2: TYPE="squashfs"
/dev/loop0: TYPE="squashfs"
/dev/loop9: TYPE="squashfs"
/dev/loop7: TYPE="squashfs"
/dev/sda2: UUID="614D-731C" BLOCK_SIZE="512" TYPE="vfat" PARTLABEL="EFI System Partition" PARTUUID="d085861b-1c74-40e3-97a2-612c45c3a626"
/dev/sda1: PARTUUID="38223da1-ce76-41e4-9ef9-5be4c935a26f"
/dev/loop5: TYPE="squashfs"
/dev/loop12: TYPE="squashfs"
/dev/sr1: BLOCK_SIZE="2048" UUID="2023-08-08-01-19-05-00" LABEL="Ubuntu 22.04.3 LTS amd64" TYPE="iso9660" PTTYPE="PMBR"
/dev/loop3: TYPE="squashfs"
/dev/loop10: TYPE="squashfs"
```

Verify that the partitions you created are labeled correctly.

```shell
/dev/sdb2: UUID="53580d06-65f6-4853-8a18-2bf64f74a0a0" BLOCK_SIZE="4096" TYPE="ext4" PARTUUID="886d0b38-02"
/dev/sdb1: UUID="2EF0-D8A9" BLOCK_SIZE="512" TYPE="vfat" PARTUUID="886d0b38-01"
```
*Everything seems to be as expected.*

There is another piece of information in the partition metadata. What does it do?

We can see it has a PARTUUID (Partition Universally Unique Identifier). The PARTUUID is generated based on the disk's
unique identifier and the partition number, and it is used to uniquely identify partitions on a disk.


4. An older way for the kernel to find out which file system to activate is the file /etc/fstab. This file lists all
the file systems that should be mounted when the system boots. It indicates the special file that represents the
partition, the directory where it should be mounted (the mount point), and the file system to activate.

List the content of /etc/fstab.
```shell
ben@ben-virtual-machine:~/Desktop$ cat /etc/fstab
# /etc/fstab: static file system information.
#
# Use 'blkid' to print the universally unique identifier for a
# device; this may be used with UUID= as a more robust way to name devices
# that works even if disks are added and removed. See fstab(5).
#
# <file system> <mount point>   <type>  <options>       <dump>  <pass>
# / was on /dev/sda3 during installation
UUID=b2a1a2f6-0070-45c6-ba4c-29595b38ffad /               ext4    errors=remount-ro 0       1
# /boot/efi was on /dev/sda2 during installation
UUID=614D-731C  /boot/efi       vfat    umask=0077      0       1
/swapfile                                 none            swap    sw              0       0
/dev/fd0        /media/floppy0  auto    rw,user,noauto,exec,utf8 0       0
```
What line is responsible for mounting the root (/) file system?
```shell
UUID=b2a1a2f6-0070-45c6-ba4c-29595b38ffad /               ext4    errors=remount-ro 0       1
```

This line has a particular way of referencing the partition, how?

*It uses the PARTUUID to reference the root patition. This way it ensures that the correct partition is mounted as the
root file system during system boot.*

## TASK 5: CREATE A FILE SYSTEM IN A FILE

1. Create a 100 MB file using dd:
```shell
ubuntu@primary:~$ dd if=/dev/zero of=/tmp/bigfile bs=1M count=100
100+0 records in
100+0 records out
104857600 bytes (105 MB, 100 MiB) copied, 0.0925215 s, 1.1 GB/s
```

2. Find the next available loopback device:
```shell
ubuntu@primary:~$ losetup -f
/dev/loop5
```

3. Associate the loopback device with the file:
```shell
ubuntu@primary:~$ sudo losetup /dev/loop5 /tmp/bigfile
```

4. Verify that the association is OK:
```shell
ubuntu@primary:~$ losetup -a
/dev/loop1: []: (/var/lib/snapd/snaps/multipass-sshfs_147.snap)
/dev/loop6: []: (/var/lib/snapd/snaps/snapd_20298.snap)
/dev/loop4: []: (/var/lib/snapd/snaps/core20_2019.snap)
/dev/loop2: []: (/var/lib/snapd/snaps/bare_5.snap)
/dev/loop0: []: (/var/lib/snapd/snaps/snapd_20102.snap)
/dev/loop5: []: (/tmp/bigfile)
/dev/loop3: []: (/var/lib/snapd/snaps/lxd_24326.snap)
```

5. Create an ext4 file system on block device /dev/loop6. Create a mountpoint in /mnt/bigfile. Mount the file system on
the mountpoint. How does findmnt show the new file system?

Creation of the ext4 file system on '/dev/loop5':
```shell
ubuntu@primary:~$ sudo mkfs.ext4 /dev/loop5
mke2fs 1.46.5 (30-Dec-2021)
Discarding device blocks: done                            
Creating filesystem with 25600 4k blocks and 25600 inodes

Allocating group tables: done                            
Writing inode tables: done                            
Creating journal (1024 blocks): done
Writing superblocks and filesystem accounting information: done
```

Creation of the mount point '/mnt/bigfile':
```shell
ubuntu@primary:~$ sudo mkdir -p /mnt/bigfile
```

Mounting the ext4 file system on the mount point:
```shell
ubuntu@primary:~$ sudo mount /dev/loop5 /mnt/bigfile
```

Checking how 'findmnt' shows the new file system:
```shell
ubuntu@primary:~$ findmnt /mnt/bigfile
TARGET       SOURCE     FSTYPE OPTIONS
/mnt/bigfile /dev/loop5 ext4   rw,relatime
```

6. Create a few files in the file system with unique strings. By searching the content of bigfile, can you find the
strings? Use the sync command to force the kernel to write buffered data to disk.

Creation of unique Strings:
```shell
ubuntu@primary:~$ echo "Test 123" > /mnt/bigfile/file1.txt
ubuntu@primary:~$ echo "Quick brown fox" > /mnt/bigfile/file2.txt
```

We can find the Strings created previously:
```shell
ubuntu@primary:~$ sudo grep -rl "Test" /mnt/bigfile
/mnt/bigfile/file1.txt

ubuntu@primary:~$ sudo grep -rl "fox" /mnt/bigfile
/mnt/bigfile/file2.txt
```

Command to force the kernel to write buffered data to disk:
```shell
ubuntu@primary:~$ sync
```

7. Undo everything:

Unmounting the file system:
```shell
ubuntu@primary:~$ sudo umount /mnt/bigfile
```

Freeing the Loopback Device:
```shell
ubuntu@primary:~$ sudo losetup -d /dev/loop5
```

Verifying that the association is no longer there:
```shell
ubuntu@primary:~$ losetup -a
/dev/loop1: []: (/var/lib/snapd/snaps/multipass-sshfs_147.snap)
/dev/loop6: []: (/var/lib/snapd/snaps/snapd_20298.snap)
/dev/loop4: []: (/var/lib/snapd/snaps/core20_2019.snap)
/dev/loop2: []: (/var/lib/snapd/snaps/bare_5.snap)
/dev/loop0: []: (/var/lib/snapd/snaps/snapd_20102.snap)
/dev/loop3: []: (/var/lib/snapd/snaps/lxd_24326.snap)
```
