# ProfHW2
## Домашнее задание №2. Работа с mdadm.

Для выполнение ДЗ, я добавил 5 виртуальных блочных устройств.
```
root@UbuntuTestVirt:~# lsblk
NAME                      MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda                         8:0    0 19,5G  0 disk
├─sda1                      8:1    0    1M  0 part
├─sda2                      8:2    0  1,8G  0 part /boot
└─sda3                      8:3    0 17,7G  0 part
  └─ubuntu--vg-ubuntu--lv 252:0    0   10G  0 lvm  /
sdb                         8:16   0    1G  0 disk
sdc                         8:32   0    1G  0 disk
sdd                         8:48   0    1G  0 disk
sde                         8:64   0    1G  0 disk
sdf                         8:80   0    1G  0 disk
sr0                        11:0    1 1024M  0 rom
```
***
Диски не подмонтированы:
```
root@UbuntuTestVirt:~# df -h
Filesystem                         Size  Used Avail Use% Mounted on
tmpfs                              197M  1,1M  196M   1% /run
/dev/mapper/ubuntu--vg-ubuntu--lv  9,8G  5,1G  4,2G  55% /
tmpfs                              984M     0  984M   0% /dev/shm
tmpfs                              5,0M     0  5,0M   0% /run/lock
/dev/sda2                          1,7G  185M  1,5G  12% /boot
tmpfs                              197M   12K  197M   1% /run/user/1000
```  
raid-массивов нет:
```
root@UbuntuTestVirt:~# cat /proc/mdstat
Personalities : [raid10] [raid0] [raid1] [raid6] [raid5] [raid4]
unused devices: <none>
```
***
Создаём raid-массив 10 из 4ёх блочных устройств sdb, sdc, sdd, sde:
```
root@UbuntuTestVirt:~# mdadm --create --verbose /dev/md127 -l 10 -n 4 /dev/sd{b..e}
root@UbuntuTestVirt:~# mdadm --create --verbose /dev/md127 -l 10 -n 4 /dev/sd{b..e}
mdadm: layout defaults to n2
mdadm: layout defaults to n2
mdadm: chunk size defaults to 512K
mdadm: /dev/sdb appears to contain an ext2fs file system
       size=1048576K  mtime=Fri May 16 00:02:40 2025
mdadm: size set to 1046528K
Continue creating array? y
mdadm: Defaulting to version 1.2 metadata
mdadm: array /dev/md127 started.
```
***
Проверяем состояние raid-массива:
```
root@UbuntuTestVirt:~# cat /proc/mdstat
Personalities : [raid0] [raid1] [raid6] [raid5] [raid4] [raid10]
md127 : active raid10 sde[3] sdd[2] sdc[1] sdb[0]
      2093056 blocks super 1.2 512K chunks 2 near-copies [4/4] [UUUU]

unused devices: <none>
```
```
root@UbuntuTestVirt:~# mdadm -D /dev/md127
/dev/md127:
           Version : 1.2
     Creation Time : Sun May 18 00:19:57 2025
        Raid Level : raid10
        Array Size : 2093056 (2044.00 MiB 2143.29 MB)
     Used Dev Size : 1046528 (1022.00 MiB 1071.64 MB)
      Raid Devices : 4
     Total Devices : 4
       Persistence : Superblock is persistent

       Update Time : Sun May 18 01:00:08 2025
             State : clean
    Active Devices : 4
   Working Devices : 4
    Failed Devices : 0
     Spare Devices : 0

            Layout : near=2
        Chunk Size : 512K

Consistency Policy : resync

              Name : UbuntuTestVirt:127  (local to host UbuntuTestVirt)
              UUID : 5cfa6e25:4d6d9b0f:52b68bae:547c05d9
            Events : 17

    Number   Major   Minor   RaidDevice State
       0       8       16        0      active sync set-A   /dev/sdb
       1       8       32        1      active sync set-B   /dev/sdc
       2       8       48        2      active sync set-A   /dev/sdd
       3       8       64        3      active sync set-B   /dev/sde
```
***
Создаём файловую систему на raid-массиве:
```
root@UbuntuTestVirt:~# mkfs.ext4 /dev/md127
mke2fs 1.47.0 (5-Feb-2023)
/dev/md127 contains a ext4 file system
        last mounted on Fri May 16 17:53:43 2025
Proceed anyway? (y,N) y
Creating filesystem with 523264 4k blocks and 130816 inodes
Filesystem UUID: 59d345ae-53e9-41b1-95c0-67e5394138a4
Superblock backups stored on blocks:
        32768, 98304, 163840, 229376, 294912

Allocating group tables: done
Writing inode tables: done
Creating journal (8192 blocks): done
Writing superblocks and filesystem accounting information: done
```
***
Монтируем raid-массив и наполняем его данными (для достоверности результатов когда мы будем ломать raid):
```
root@UbuntuTestVirt:~# mount /dev/md127 /mnt/raid/
root@UbuntuTestVirt:~# df -h
Filesystem                         Size  Used Avail Use% Mounted on
tmpfs                              197M  1,1M  196M   1% /run
/dev/mapper/ubuntu--vg-ubuntu--lv  9,8G  5,1G  4,2G  55% /
tmpfs                              984M     0  984M   0% /dev/shm
tmpfs                              5,0M     0  5,0M   0% /run/lock
/dev/sda2                          1,7G  185M  1,5G  12% /boot
tmpfs                              197M   12K  197M   1% /run/user/1000
/dev/md127                         2,0G   24K  1,9G   1% /mnt/raid
```
Копируем данные.
Смотрим изменение размера данных на /dev/md127
```
root@UbuntuTestVirt:~# df -h
...
/dev/md127                         2,0G  1,1G  739M  61% /mnt/raid
```
Содержимое /mnt/raid/.
<details>
  <summary>Вывод соддержимого /mnt/raid/</summary>
    <code>
      root@UbuntuTestVirt:~# ls /mnt/raid/
      01.mp3  08.mp3  15.mp3                 auth.log.1     cloud-init.log         dmesg.3.gz     installer      landscape   syslog.1                       wtmp    
      02.mp3  09.mp3  alternatives.log       auth.log.2.gz  cloud-init-output.log  dmesg.4.gz     journal        lastlog     syslog.2.gz
      03.mp3  10.mp3  alternatives.log.1     auth.log.3.gz  dist-upgrade           dpkg.log       kern.log       log         syslog.3.gz
      04.mp3  11.mp3  alternatives.log.2.gz  auth.log.4.gz  dmesg                  dpkg.log.1     kern.log.1     lost+found  syslog.4.gz
      05.mp3  12.mp3  apport.log             bootstrap.log  dmesg.0                dpkg.log.2.gz  kern.log.2.gz  private     sysstat
      06.mp3  13.mp3  apt                    btmp           dmesg.1.gz             dpkg.log.3.gz  kern.log.3.gz  README      ubuntu-advantage-apt-hook.log
      07.mp3  14.mp3  auth.log               btmp.1         dmesg.2.gz             faillog        kern.log.4.gz  syslog      unattended-upgrades
    </code>
</details><br>
---
"Ломаем" raid-массив.
<code>
root@UbuntuTestVirt:~# mdadm -D /dev/md127
/dev/md127:
           Version : 1.2
     Creation Time : Sun May 18 00:19:57 2025
        Raid Level : raid10
        Array Size : 2093056 (2044.00 MiB 2143.29 MB)
     Used Dev Size : 1046528 (1022.00 MiB 1071.64 MB)
      Raid Devices : 4
     Total Devices : 4
       Persistence : Superblock is persistent

       Update Time : Sun May 18 01:33:20 2025
             State : clean
    Active Devices : 4
   Working Devices : 4
    Failed Devices : 0
     Spare Devices : 0

            Layout : near=2
        Chunk Size : 512K

Consistency Policy : resync

              Name : UbuntuTestVirt:127  (local to host UbuntuTestVirt)
              UUID : 5cfa6e25:4d6d9b0f:52b68bae:547c05d9
            Events : 17

    Number   Major   Minor   RaidDevice State
       0       8       16        0      active sync set-A   /dev/sdb
       1       8       32        1      active sync set-B   /dev/sdc
       2       8       48        2      active sync set-A   /dev/sdd
       3       8       64        3      active sync set-B   /dev/sde  
</code>
```
root@UbuntuTestVirt:~# mdadm /dev/md127 --fail /dev/sdd
mdadm: set /dev/sdd faulty in /dev/md127
```
```
root@UbuntuTestVirt:~# mdadm -D /dev/md127
/dev/md127:
           Version : 1.2
     Creation Time : Sun May 18 00:19:57 2025
        Raid Level : raid10
        Array Size : 2093056 (2044.00 MiB 2143.29 MB)
     Used Dev Size : 1046528 (1022.00 MiB 1071.64 MB)
      Raid Devices : 4
     Total Devices : 4
       Persistence : Superblock is persistent

       Update Time : Sun May 18 22:29:54 2025
             State : clean, degraded
    Active Devices : 3
   Working Devices : 3
    Failed Devices : 1
     Spare Devices : 0

            Layout : near=2
        Chunk Size : 512K

Consistency Policy : resync

              Name : UbuntuTestVirt:127  (local to host UbuntuTestVirt)
              UUID : 5cfa6e25:4d6d9b0f:52b68bae:547c05d9
            Events : 19

    Number   Major   Minor   RaidDevice State
       0       8       16        0      active sync set-A   /dev/sdb
       1       8       32        1      active sync set-B   /dev/sdc
       -       0        0        2      removed
       3       8       64        3      active sync set-B   /dev/sde

       2       8       48        -      faulty   /dev/sdd
```
...
Проверяем сохранность данных.
<details>
  <summary>Вывод соддержимого /mnt/raid/</summary>
  <code>root@UbuntuTestVirt:~# ls /mnt/raid/
01.mp3  08.mp3  15.mp3                 auth.log.1     cloud-init.log         dmesg.3.gz     installer      landscape   syslog.1                       wtmp
02.mp3  09.mp3  alternatives.log       auth.log.2.gz  cloud-init-output.log  dmesg.4.gz     journal        lastlog     syslog.2.gz
03.mp3  10.mp3  alternatives.log.1     auth.log.3.gz  dist-upgrade           dpkg.log       kern.log       log         syslog.3.gz
04.mp3  11.mp3  alternatives.log.2.gz  auth.log.4.gz  dmesg                  dpkg.log.1     kern.log.1     lost+found  syslog.4.gz
05.mp3  12.mp3  apport.log             bootstrap.log  dmesg.0                dpkg.log.2.gz  kern.log.2.gz  private     sysstat
06.mp3  13.mp3  apt                    btmp           dmesg.1.gz             dpkg.log.3.gz  kern.log.3.gz  README      ubuntu-advantage-apt-hook.log
07.mp3  14.mp3  auth.log               btmp.1         dmesg.2.gz             faillog        kern.log.4.gz  syslog      unattended-upgrades</code>
</details>
...
Извлекаем "сбойный диск"
```
root@UbuntuTestVirt:~# mdadm /dev/md127 --remove /dev/sdd
mdadm: hot removed /dev/sdd from /dev/md127
```
```
root@UbuntuTestVirt:~# mdadm -D /dev/md127
/dev/md127:
           Version : 1.2
     Creation Time : Sun May 18 00:19:57 2025
        Raid Level : raid10
        Array Size : 2093056 (2044.00 MiB 2143.29 MB)
     Used Dev Size : 1046528 (1022.00 MiB 1071.64 MB)
      Raid Devices : 4
     Total Devices : 3
       Persistence : Superblock is persistent

       Update Time : Sun May 18 23:34:30 2025
             State : clean, degraded
    Active Devices : 3
   Working Devices : 3
    Failed Devices : 0
     Spare Devices : 0

            Layout : near=2
        Chunk Size : 512K

Consistency Policy : resync

              Name : UbuntuTestVirt:127  (local to host UbuntuTestVirt)
              UUID : 5cfa6e25:4d6d9b0f:52b68bae:547c05d9
            Events : 28

    Number   Major   Minor   RaidDevice State
       0       8       16        0      active sync set-A   /dev/sdb
       1       8       32        1      active sync set-B   /dev/sdc
       -       0        0        2      removed
       3       8       64        3      active sync set-B   /dev/sdd
```
...
Добаваляем в raid-массив новый диск.
```
root@UbuntuTestVirt:~# mdadm /dev/md127 --add /dev/sdf
mdadm: added /dev/sdf
```
```
root@UbuntuTestVirt:~# mdadm -D /dev/md127
/dev/md127:
           Version : 1.2
     Creation Time : Sun May 18 00:19:57 2025
        Raid Level : raid10
        Array Size : 2093056 (2044.00 MiB 2143.29 MB)
     Used Dev Size : 1046528 (1022.00 MiB 1071.64 MB)
      Raid Devices : 4
     Total Devices : 4
       Persistence : Superblock is persistent

       Update Time : Sun May 18 23:39:40 2025
             State : clean, degraded, recovering
    Active Devices : 3
   Working Devices : 4
    Failed Devices : 0
     Spare Devices : 1

            Layout : near=2
        Chunk Size : 512K

Consistency Policy : resync

    Rebuild Status : 14% complete

              Name : UbuntuTestVirt:127  (local to host UbuntuTestVirt)
              UUID : 5cfa6e25:4d6d9b0f:52b68bae:547c05d9
            Events : 34

    Number   Major   Minor   RaidDevice State
       0       8       16        0      active sync set-A   /dev/sdb
       1       8       32        1      active sync set-B   /dev/sdc
       4       8       80        2      spare rebuilding   /dev/sdf
       3       8       64        3      active sync set-B   /dev/sde
```
```
root@UbuntuTestVirt:~# mdadm -D /dev/md127
/dev/md127:
           Version : 1.2
     Creation Time : Sun May 18 00:19:57 2025
        Raid Level : raid10
        Array Size : 2093056 (2044.00 MiB 2143.29 MB)
     Used Dev Size : 1046528 (1022.00 MiB 1071.64 MB)
      Raid Devices : 4
     Total Devices : 4
       Persistence : Superblock is persistent

       Update Time : Sun May 18 23:40:07 2025
             State : clean
    Active Devices : 4
   Working Devices : 4
    Failed Devices : 0
     Spare Devices : 0

            Layout : near=2
        Chunk Size : 512K

Consistency Policy : resync

              Name : UbuntuTestVirt:127  (local to host UbuntuTestVirt)
              UUID : 5cfa6e25:4d6d9b0f:52b68bae:547c05d9
            Events : 49

    Number   Major   Minor   RaidDevice State
       0       8       16        0      active sync set-A   /dev/sdb
       1       8       32        1      active sync set-B   /dev/sdc
       4       8       80        2      active sync set-A   /dev/sdf
       3       8       64        3      active sync set-B   /dev/sde
```
...
raid-массив восстановлен!
...
