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

2. Convince yourself that the special file representing the root (/) partition can be read just like any other file.
As it contains binary data, just opening it with less will mess up the terminal, so use the xxd hexdump utility.


