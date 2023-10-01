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



