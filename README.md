# ProfHW2
Домашнее задание №2. Работа с mdadm.

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
Запускаем скрипт для создания raid-массива 10 из 4ёх дисков c файловой сиситемой ext4:
