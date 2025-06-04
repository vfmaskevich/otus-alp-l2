# Занятие 2. Дисковая подсистема
## Задание
### Добавить в виртуальную машину несколько дисков
Собрать RAID-0/1/5/10 на выбор
Сломать и починить RAID
Создать GPT таблицу, пять разделов и смонтировать их в системе.

### На проверку отправьте:
скрипт для создания рейда, 
отчет по командам для починки RAID и созданию разделов.

### Добавить в виртуальную машину несколько дисков
В настройках виртуальной машины нужно добавить несколько дисков одинакового размера, для учебных задач достаточны диски по 1GB.
Далее подразумеваем, что мы добавили в виртуальную машину 5 дисков.

### Собрать RAID-0/1/5/10 — на выбор
Далее нужно определиться, какого уровня RAID будем собирать. Для этого посмотрим, какие блочные устройства у нас есть и исходя из их количества, размера и поставленной задачи, определимся.

Команды:

```bash
# Cмотрим, какие блочные устройства у нас есть
vladimir@ubuntu-server-vm:~$ sudo lsblk
[sudo] password for vladimir:
NAME                      MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
loop0                       7:0    0 63.9M  1 loop /snap/core20/2318
loop1                       7:1    0   87M  1 loop /snap/lxd/29351
loop2                       7:2    0 38.8M  1 loop /snap/snapd/21759
sda                         8:0    0   40G  0 disk
├─sda1                      8:1    0    1M  0 part
├─sda2                      8:2    0    2G  0 part /boot
└─sda3                      8:3    0   38G  0 part
  └─ubuntu--vg-ubuntu--lv 253:0    0   19G  0 lvm  /
sdb                         8:16   0    1G  0 disk
sdc                         8:32   0    1G  0 disk
sdd                         8:48   0    1G  0 disk
sde                         8:64   0    1G  0 disk
sr0                        11:0    1 1024M  0 rom

vladimir@ubuntu-server-vm:~$ sudo sudo lshw -short | grep disk
/0/100/1.1/0.0.0  /dev/cdrom  disk        CD-ROM
/0/100/d/0        /dev/sda    disk        42GB VBOX HARDDISK
/0/100/d/1        /dev/sdb    disk        1073MB VBOX HARDDISK
/0/100/d/2        /dev/sdc    disk        1073MB VBOX HARDDISK
/0/100/d/3        /dev/sdd    disk        1073MB VBOX HARDDISK
/0/100/d/0.0.0    /dev/sde    disk        1073MB VBOX HARDDISK

vladimir@ubuntu-server-vm:~$ sudo fdisk -l
Disk /dev/loop0: 63.95 MiB, 67051520 bytes, 130960 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes


Disk /dev/loop1: 87.04 MiB, 91267072 bytes, 178256 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes


Disk /dev/loop2: 38.83 MiB, 40714240 bytes, 79520 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes


Disk /dev/sda: 40 GiB, 42949672960 bytes, 83886080 sectors
Disk model: VBOX HARDDISK
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt
Disk identifier: E4203279-CF56-4E12-8996-845B2CC447ED

Device       Start      End  Sectors Size Type
/dev/sda1     2048     4095     2048   1M BIOS boot
/dev/sda2     4096  4198399  4194304   2G Linux filesystem
/dev/sda3  4198400 83884031 79685632  38G Linux filesystem


Disk /dev/sdb: 1 GiB, 1073741824 bytes, 2097152 sectors
Disk model: VBOX HARDDISK
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes


Disk /dev/sdc: 1 GiB, 1073741824 bytes, 2097152 sectors
Disk model: VBOX HARDDISK
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes


Disk /dev/sdd: 1 GiB, 1073741824 bytes, 2097152 sectors
Disk model: VBOX HARDDISK
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes


Disk /dev/sde: 1 GiB, 1073741824 bytes, 2097152 sectors
Disk model: VBOX HARDDISK
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes


Disk /dev/mapper/ubuntu--vg-ubuntu--lv: 19 GiB, 20396900352 bytes, 39837696 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes

# Занулим на всякий случай суперблоки:
vladimir@ubuntu-server-vm:~$ sudo mdadm --zero-superblock --force /dev/sd{b,c,d,e}
mdadm: Unrecognised md component device - /dev/sdb
mdadm: Unrecognised md component device - /dev/sdc
mdadm: Unrecognised md component device - /dev/sdd
mdadm: Unrecognised md component device - /dev/sde


# Cоздаtv RAID-5 из 4 дисков:
vladimir@ubuntu-server-vm:~$ sudo mdadm --create --verbose /dev/md0 -l 5 -n 4 /dev/sd{b,c,d,e}
mdadm: layout defaults to left-symmetric
mdadm: layout defaults to left-symmetric
mdadm: chunk size defaults to 512K
mdadm: size set to 1046528K
mdadm: Defaulting to version 1.2 metadata
mdadm: array /dev/md0 started.

# Проверим, что RAID собрался нормально:
vladimir@ubuntu-server-vm:~$ cat /proc/mdstat
Personalities : [linear] [multipath] [raid0] [raid1] [raid6] [raid5] [raid4] [raid10]
md0 : active raid5 sde[4] sdd[2] sdc[1] sdb[0]
      3139584 blocks super 1.2 level 5, 512k chunk, algorithm 2 [4/4] [UUUU]

unused devices: <none>

vladimir@ubuntu-server-vm:~$ sudo mdadm -D /dev/md0
/dev/md0:
           Version : 1.2
     Creation Time : Wed Jun  4 11:37:44 2025
        Raid Level : raid5
        Array Size : 3139584 (2.99 GiB 3.21 GB)
     Used Dev Size : 1046528 (1022.00 MiB 1071.64 MB)
      Raid Devices : 4
     Total Devices : 4
       Persistence : Superblock is persistent

       Update Time : Wed Jun  4 11:38:01 2025
             State : clean
    Active Devices : 4
   Working Devices : 4
    Failed Devices : 0
     Spare Devices : 0

            Layout : left-symmetric
        Chunk Size : 512K

Consistency Policy : resync

              Name : ubuntu-server-vm:0  (local to host ubuntu-server-vm)
              UUID : 017e3a3a:c7521803:7d976931:f02a1273
            Events : 18

    Number   Major   Minor   RaidDevice State
       0       8       16        0      active sync   /dev/sdb
       1       8       32        1      active sync   /dev/sdc
       2       8       48        2      active sync   /dev/sdd
       4       8       64        3      active sync   /dev/sde

# Сломать и починить RAID
vladimir@ubuntu-server-vm:~$ sudo mdadm /dev/md0 --fail /dev/sdd
mdadm: set /dev/sdd faulty in /dev/md0

# Посмотрим, как это отразилось на RAID:
vladimir@ubuntu-server-vm:~$ cat /proc/mdstat
Personalities : [linear] [multipath] [raid0] [raid1] [raid6] [raid5] [raid4] [raid10]
md0 : active raid5 sde[4] sdd[2](F) sdc[1] sdb[0]
      3139584 blocks super 1.2 level 5, 512k chunk, algorithm 2 [4/3] [UU_U]

unused devices: <none>

vladimir@ubuntu-server-vm:~$ sudo mdadm -D /dev/md0
/dev/md0:
           Version : 1.2
     Creation Time : Wed Jun  4 11:37:44 2025
        Raid Level : raid5
        Array Size : 3139584 (2.99 GiB 3.21 GB)
     Used Dev Size : 1046528 (1022.00 MiB 1071.64 MB)
      Raid Devices : 4
     Total Devices : 4
       Persistence : Superblock is persistent

       Update Time : Wed Jun  4 11:40:47 2025
             State : clean, degraded
    Active Devices : 3
   Working Devices : 3
    Failed Devices : 1
     Spare Devices : 0

            Layout : left-symmetric
        Chunk Size : 512K

Consistency Policy : resync

              Name : ubuntu-server-vm:0  (local to host ubuntu-server-vm)
              UUID : 017e3a3a:c7521803:7d976931:f02a1273
            Events : 20

    Number   Major   Minor   RaidDevice State
       0       8       16        0      active sync   /dev/sdb
       1       8       32        1      active sync   /dev/sdc
       -       0        0        2      removed
       4       8       64        3      active sync   /dev/sde

       2       8       48        -      faulty   /dev/sdd

# Удалим “сломанный” диск из массива:
vladimir@ubuntu-server-vm:~$ sudo mdadm /dev/md0 --remove /dev/sdd
mdadm: hot removed /dev/sdd from /dev/md0

# Представим, что мы вставили новый диск в сервер и теперь нам нужно добавить его в RAID.
vladimir@ubuntu-server-vm:~$ sudo mdadm /dev/md0 --add /dev/sdd
mdadm: added /dev/sdd

# Процесс rebuild-а можно увидеть в выводе следующих команд:
vladimir@ubuntu-server-vm:~$ cat /proc/mdstat
Personalities : [linear] [multipath] [raid0] [raid1] [raid6] [raid5] [raid4] [raid10]
md0 : active raid5 sdd[5] sde[4] sdc[1] sdb[0]
      3139584 blocks super 1.2 level 5, 512k chunk, algorithm 2 [4/4] [UUUU]

unused devices: <none>

vladimir@ubuntu-server-vm:~$ sudo mdadm -D /dev/md0
/dev/md0:
           Version : 1.2
     Creation Time : Wed Jun  4 11:37:44 2025
        Raid Level : raid5
        Array Size : 3139584 (2.99 GiB 3.21 GB)
     Used Dev Size : 1046528 (1022.00 MiB 1071.64 MB)
      Raid Devices : 4
     Total Devices : 4
       Persistence : Superblock is persistent

       Update Time : Wed Jun  4 11:43:35 2025
             State : clean
    Active Devices : 4
   Working Devices : 4
    Failed Devices : 0
     Spare Devices : 0

            Layout : left-symmetric
        Chunk Size : 512K

Consistency Policy : resync

              Name : ubuntu-server-vm:0  (local to host ubuntu-server-vm)
              UUID : 017e3a3a:c7521803:7d976931:f02a1273
            Events : 40

    Number   Major   Minor   RaidDevice State
       0       8       16        0      active sync   /dev/sdb
       1       8       32        1      active sync   /dev/sdc
       5       8       48        2      active sync   /dev/sdd
       4       8       64        3      active sync   /dev/sde

# Создать GPT таблицу, 5 разделов и смонтировать их в системе
# Создаем раздел GPT на RAID
vladimir@ubuntu-server-vm:~$ sudo parted -s /dev/md0 mklabel gpt

# Создаем партиции
vladimir@ubuntu-server-vm:~$ sudo parted /dev/md0 mkpart primary ext4 0% 20%
Information: You may need to update /etc/fstab.

vladimir@ubuntu-server-vm:~$ sudo parted /dev/md0 mkpart primary ext4 20% 40%
Information: You may need to update /etc/fstab.

vladimir@ubuntu-server-vm:~$ sudo parted /dev/md0 mkpart primary ext4 40% 60%
Information: You may need to update /etc/fstab.

vladimir@ubuntu-server-vm:~$ sudo parted /dev/md0 mkpart primary ext4 60% 80%
Information: You may need to update /etc/fstab.

vladimir@ubuntu-server-vm:~$ sudo parted /dev/md0 mkpart primary ext4 80% 100%
Information: You may need to update /etc/fstab.

# Далее можно создать на этих партициях ФС
vladimir@ubuntu-server-vm:~$ for i in $(seq 1 5); do sudo mkfs.ext4 /dev/md0p$i; done
mke2fs 1.46.5 (30-Dec-2021)
/dev/md0p1 contains a ext4 file system
        created on Wed Jun  4 10:36:55 2025
Proceed anyway? (y,N) y
Creating filesystem with 156672 4k blocks and 39200 inodes
Filesystem UUID: bc64e061-d081-4c37-9d7d-b8c92fe9edb8
Superblock backups stored on blocks:
        32768, 98304

Allocating group tables: done
Writing inode tables: done
Creating journal (4096 blocks): done
Writing superblocks and filesystem accounting information: done

mke2fs 1.46.5 (30-Dec-2021)
/dev/md0p2 contains a ext4 file system
        created on Wed Jun  4 10:36:55 2025
Proceed anyway? (y,N) y
Creating filesystem with 157056 4k blocks and 39280 inodes
Filesystem UUID: 166834b0-31a1-4bec-a7d9-849b3fdc2d84
Superblock backups stored on blocks:
        32768, 98304

Allocating group tables: done
Writing inode tables: done
Creating journal (4096 blocks): done
Writing superblocks and filesystem accounting information: done

mke2fs 1.46.5 (30-Dec-2021)
/dev/md0p3 contains a ext4 file system
        created on Wed Jun  4 10:36:56 2025
Proceed anyway? (y,N) y
Creating filesystem with 156672 4k blocks and 39200 inodes
Filesystem UUID: 9139e745-8c1d-41dd-921e-f13447599425
Superblock backups stored on blocks:
        32768, 98304

Allocating group tables: done
Writing inode tables: done
Creating journal (4096 blocks): done
Writing superblocks and filesystem accounting information: done

mke2fs 1.46.5 (30-Dec-2021)
Creating filesystem with 157056 4k blocks and 39280 inodes
Filesystem UUID: 60431d7e-ce6b-4033-95ad-91eb138acac0
Superblock backups stored on blocks:
        32768, 98304

Allocating group tables: done
Writing inode tables: done
Creating journal (4096 blocks): done
Writing superblocks and filesystem accounting information: done

mke2fs 1.46.5 (30-Dec-2021)
/dev/md0p5 contains a ext4 file system
        created on Wed Jun  4 10:36:56 2025
Proceed anyway? (y,N) y
Creating filesystem with 156672 4k blocks and 39200 inodes
Filesystem UUID: 2f6c61e9-0f99-42ec-8430-4dbf4a9304c7
Superblock backups stored on blocks:
        32768, 98304

Allocating group tables: done
Writing inode tables: done
Creating journal (4096 blocks): done
Writing superblocks and filesystem accounting information: done

# И смонтировать их по каталогам
vladimir@ubuntu-server-vm:~$ sudo su
root@ubuntu-server-vm:/home/vladimir# for i in $(seq 1 5); do mount /dev/md0p$i /raid/part$i; done

# Смотрим результат
root@ubuntu-server-vm:/home/vladimir# lsblk
NAME                      MAJ:MIN RM   SIZE RO TYPE  MOUNTPOINTS
loop0                       7:0    0  63.9M  1 loop  /snap/core20/2318
loop1                       7:1    0    87M  1 loop  /snap/lxd/29351
loop2                       7:2    0  38.8M  1 loop  /snap/snapd/21759
loop3                       7:3    0  50.9M  1 loop  /snap/snapd/24505
loop4                       7:4    0  63.8M  1 loop  /snap/core20/2582
loop5                       7:5    0  89.4M  1 loop  /snap/lxd/31333
sda                         8:0    0    40G  0 disk
├─sda1                      8:1    0     1M  0 part
├─sda2                      8:2    0     2G  0 part  /boot
└─sda3                      8:3    0    38G  0 part
  └─ubuntu--vg-ubuntu--lv 253:0    0    19G  0 lvm   /
sdb                         8:16   0     1G  0 disk
└─md0                       9:0    0     3G  0 raid5
  ├─md0p5                 259:0    0   612M  0 part  /raid/part5
  ├─md0p1                 259:4    0   612M  0 part  /raid/part1
  ├─md0p2                 259:5    0 613.5M  0 part  /raid/part2
  ├─md0p3                 259:6    0   612M  0 part  /raid/part3
  └─md0p4                 259:7    0 613.5M  0 part  /raid/part4
sdc                         8:32   0     1G  0 disk
└─md0                       9:0    0     3G  0 raid5
  ├─md0p5                 259:0    0   612M  0 part  /raid/part5
  ├─md0p1                 259:4    0   612M  0 part  /raid/part1
  ├─md0p2                 259:5    0 613.5M  0 part  /raid/part2
  ├─md0p3                 259:6    0   612M  0 part  /raid/part3
  └─md0p4                 259:7    0 613.5M  0 part  /raid/part4
sdd                         8:48   0     1G  0 disk
└─md0                       9:0    0     3G  0 raid5
  ├─md0p5                 259:0    0   612M  0 part  /raid/part5
  ├─md0p1                 259:4    0   612M  0 part  /raid/part1
  ├─md0p2                 259:5    0 613.5M  0 part  /raid/part2
  ├─md0p3                 259:6    0   612M  0 part  /raid/part3
  └─md0p4                 259:7    0 613.5M  0 part  /raid/part4
sde                         8:64   0     1G  0 disk
└─md0                       9:0    0     3G  0 raid5
  ├─md0p5                 259:0    0   612M  0 part  /raid/part5
  ├─md0p1                 259:4    0   612M  0 part  /raid/part1
  ├─md0p2                 259:5    0 613.5M  0 part  /raid/part2
  ├─md0p3                 259:6    0   612M  0 part  /raid/part3
  └─md0p4                 259:7    0 613.5M  0 part  /raid/part4
sr0                        11:0    1  1024M  0 rom


```
