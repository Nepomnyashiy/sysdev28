# Домашнее задание к занятию «Файловые системы»

## Задание

1. Узнайте о [sparse-файлах]

Разреженный файл - это файл, в котором последовательности нулевых байтов заменены на информацию об этих последовательностях (список дыр). Такой файл эффективен, потому что он не хранит нули на диске, вместо этого он содержит достаточно метаданных, описывающих нули, которые будут сгенерированы.

2. Могут ли файлы, являющиеся жёсткой ссылкой на один объект, иметь разные права доступа и владельца? Почему?

Не могут. Это ссылки на один и тот же inode - именно в нём и хранятся права доступа и имя владельца.

3. Сделайте `vagrant destroy` на имеющийся инстанс Ubuntu. Замените содержимое Vagrantfile следующим:

    ```ruby
    path_to_disk_folder = './disks'

    host_params = {
        'disk_size' => 2560,
        'disks'=>[1, 2],
        'cpus'=>2,
        'memory'=>2048,
        'hostname'=>'sysadm-fs',
        'vm_name'=>'sysadm-fs'
    }
    Vagrant.configure("2") do |config|
        config.vm.box = "bento/ubuntu-20.04"
        config.vm.hostname=host_params['hostname']
        config.vm.provider :virtualbox do |v|

            v.name=host_params['vm_name']
            v.cpus=host_params['cpus']
            v.memory=host_params['memory']

            host_params['disks'].each do |disk|
                file_to_disk=path_to_disk_folder+'/disk'+disk.to_s+'.vdi'
                unless File.exist?(file_to_disk)
                    v.customize ['createmedium', '--filename', file_to_disk, '--size', host_params['disk_size']]
                    v.customize ['storageattach', :id, '--storagectl', 'SATA Controller', '--port', disk.to_s, '--device', 0, '--type', 'hdd', '--medium', file_to_disk]
                end
            end
        end
        config.vm.network "private_network", type: "dhcp"
    end
    ```

    Эта конфигурация созfdiskдаст новую виртуальную машину с двумя дополнительными неразмеченными дисками по 2,5 Гб.

```bash
vagrant@sysadm-fs:~$ lsblk
NAME                      MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
loop0                       7:0    0 67.8M  1 loop /snap/lxd/22753
loop1                       7:1    0   62M  1 loop /snap/core20/1611
loop3                       7:3    0 53.2M  1 loop /snap/snapd/19122
loop4                       7:4    0 63.5M  1 loop /snap/core20/1891
loop5                       7:5    0 91.9M  1 loop /snap/lxd/24061
sda                         8:0    0   64G  0 disk 
├─sda1                      8:1    0    1M  0 part 
├─sda2                      8:2    0    2G  0 part /boot
└─sda3                      8:3    0   62G  0 part 
  └─ubuntu--vg-ubuntu--lv 253:0    0   31G  0 lvm  /
sdb                         8:16   0  2.5G  0 disk 
sdc                         8:32   0  2.5G  0 disk 
```

4. Используя `fdisk`, разбейте первый диск на два раздела: 2 Гб и оставшееся пространство.

```bash
vagrant@sysadm-fs:~$ sudo fdisk /dev/sdb

Welcome to fdisk (util-linux 2.34).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.

Device does not contain a recognized partition table.
Created a new DOS disklabel with disk identifier 0x37ae86ff.

Command (m for help): n
Partition type
   p   primary (0 primary, 0 extended, 4 free)
   e   extended (container for logical partitions)
Select (default p): p
Partition number (1-4, default 1): 1
First sector (2048-5242879, default 2048): 
Last sector, +/-sectors or +/-size{K,M,G,T,P} (2048-5242879, default 5242879): +2G

Created a new partition 1 of type 'Linux' and of size 2 GiB.

Command (m for help): n
Partition type
   p   primary (1 primary, 0 extended, 3 free)
   e   extended (container for logical partitions)
Select (default p): p
Partition number (2-4, default 2): 
First sector (4196352-5242879, default 4196352): 
Last sector, +/-sectors or +/-size{K,M,G,T,P} (4196352-5242879, default 5242879): 

Created a new partition 2 of type 'Linux' and of size 511 MiB.
```

5. Используя `sfdisk`, перенесите эту таблицу разделов на второй диск.

```bash
vagrant@sysadm-fs:~$ sudo sfdisk -d /dev/sdb > sdb.dump
vagrant@sysadm-fs:~$ sudo sfdisk /dev/sdc < sdb.dump
Checking that no-one is using this disk right now ... OK

Disk /dev/sdc: 2.51 GiB, 2684354560 bytes, 5242880 sectors
Disk model: VBOX HARDDISK   
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes

>>> Script header accepted.
>>> Script header accepted.
>>> Script header accepted.
>>> Script header accepted.
>>> Created a new DOS disklabel with disk identifier 0x37ae86ff.
/dev/sdc1: Created a new partition 1 of type 'Linux' and of size 2 GiB.
/dev/sdc2: Created a new partition 2 of type 'Linux' and of size 511 MiB.
/dev/sdc3: Done.

New situation:
Disklabel type: dos
Disk identifier: 0x37ae86ff

Device     Boot   Start     End Sectors  Size Id Type
/dev/sdc1          2048 4196351 4194304    2G 83 Linux
/dev/sdc2       4196352 5242879 1046528  511M 83 Linux

The partition table has been altered.
Calling ioctl() to re-read partition table.
Syncing disks.
```

6. Соберите `mdadm` RAID1 на паре разделов 2 Гб.

```bash
vagrant@sysadm-fs:~$ sudo mdadm --create /dev/md1 --level=1 --raid-devices=2 /dev/sd[bc]1
mdadm: Note: this array has metadata at the start and
    may not be suitable as a boot device.  If you plan to
    store '/boot' on this device please ensure that
    your boot-loader understands md/v1.x metadata, or use
    --metadata=0.90
Continue creating array? y
mdadm: Defaulting to version 1.2 metadata
mdadm: array /dev/md1 started.
```

7. Соберите `mdadm` RAID0 на второй паре маленьких разделов.

```bash
vagrant@sysadm-fs:~$ sudo mdadm --create /dev/md2 --level=0 --raid-devices=2 /dev/sd[bc]2
mdadm: Defaulting to version 1.2 metadata
mdadm: array /dev/md2 started.
```

8. Создайте два независимых PV на получившихся md-устройствах.

```bash
vagrant@sysadm-fs:~$ sudo pvcreate /dev/md1
  Physical volume "/dev/md1" successfully created.
vagrant@sysadm-fs:~$ sudo pvcreate /dev/md2
  Physical volume "/dev/md2" successfully created.
```

9. Создайте общую volume-group на этих двух PV.

```bash
vagrant@sysadm-fs:~$ sudo vgcreate ntlg /dev/md1 /dev/md2
  Volume group "ntlg" successfully created
```

10. Создайте LV размером 100 Мб, указав его расположение на PV с RAID0.

```bash
vagrant@sysadm-fs:~$ sudo vgcreate ntlg /dev/md1 /dev/md2
  Volume group "ntlg" successfully created
```

11. Создайте `mkfs.ext4` ФС на получившемся LV.

```bash
vagrant@sysadm-fs:~$ sudo mkfs.ext4 -L ntlg-ext4 -m 1 /dev/mapper/ntlg-ntlg--lv
mke2fs 1.45.5 (07-Jan-2020)
Creating filesystem with 25600 4k blocks and 25600 inodes

Allocating group tables: done                            
Writing inode tables: done                            
Creating journal (1024 blocks): done
Writing superblocks and filesystem accounting information: done

vagrant@sysadm-fs:~$ sudo blkid | grep ntlg-ntlg--lv
/dev/mapper/ntlg-ntlg--lv: LABEL="ntlg-ext4" UUID="27c61349-37e7-43ef-92ab-791a75708544" TYPE="ext4"
```

12. Смонтируйте этот раздел в любую директорию, например, `/tmp/new`.

```bash
vagrant@sysadm-fs:~$ mkdir /tmp/new
vagrant@sysadm-fs:~$ sudo mount /dev/mapper/ntlg-ntlg--lv /tmp/new
vagrant@sysadm-fs:~$ sudo mount | grep ntlg-ntlg--lv
/dev/mapper/ntlg-ntlg--lv on /tmp/new type ext4 (rw,relatime,stripe=256)
```

13. Поместите туда тестовый файл, например, `wget https://mirror.yandex.ru/ubuntu/ls-lR.gz -O /tmp/new/test.gz`.

```bash
vagrant@sysadm-fs:~$ cd /tmp/new
vagrant@sysadm-fs:/tmp/new$ sudo wget https://mirror.yandex.ru/ubuntu/ls-lR.gz -O /tmp/new/test.gz
--2023-05-27 16:08:54--  https://mirror.yandex.ru/ubuntu/ls-lR.gz
Resolving mirror.yandex.ru (mirror.yandex.ru)... 213.180.204.183, 2a02:6b8::183
Connecting to mirror.yandex.ru (mirror.yandex.ru)|213.180.204.183|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 25079778 (24M) [application/octet-stream]
Saving to: ‘/tmp/new/test.gz’

/tmp/new/test.gz              100%[===============================================>]  23.92M   934KB/s    in 22s     

2023-05-27 16:09:17 (1.09 MB/s) - ‘/tmp/new/test.gz’ saved [25079778/25079778]

vagrant@sysadm-fs:/tmp/new$ ls -lah
total 24M
drwxr-xr-x  3 root root 4.0K May 27 16:08 .
drwxrwxrwt 12 root root 4.0K May 27 16:03 ..
drwx------  2 root root  16K May 27 16:01 lost+found
-rw-r--r--  1 root root  24M May 27 14:58 test.gz
```

14. Прикрепите вывод `lsblk`.

```bash
vagrant@sysadm-fs:/tmp/new$ lsblk
NAME                      MAJ:MIN RM  SIZE RO TYPE  MOUNTPOINT
loop0                       7:0    0   62M  1 loop  /snap/core20/1611
loop1                       7:1    0 67.8M  1 loop  /snap/lxd/22753
loop3                       7:3    0 53.2M  1 loop  /snap/snapd/19122
loop4                       7:4    0 63.5M  1 loop  /snap/core20/1891
loop5                       7:5    0 91.9M  1 loop  /snap/lxd/24061
sda                         8:0    0   64G  0 disk  
├─sda1                      8:1    0    1M  0 part  
├─sda2                      8:2    0    2G  0 part  /boot
└─sda3                      8:3    0   62G  0 part  
  └─ubuntu--vg-ubuntu--lv 253:0    0   31G  0 lvm   /
sdb                         8:16   0  2.5G  0 disk  
├─sdb1                      8:17   0    2G  0 part  
│ └─md1                     9:1    0    2G  0 raid1 
└─sdb2                      8:18   0  511M  0 part  
  └─md2                     9:2    0 1018M  0 raid0 
    └─ntlg-ntlg--lv       253:1    0  100M  0 lvm   /tmp/new
sdc                         8:32   0  2.5G  0 disk  
├─sdc1                      8:33   0    2G  0 part  
│ └─md1                     9:1    0    2G  0 raid1 
└─sdc2                      8:34   0  511M  0 part  
  └─md2                     9:2    0 1018M  0 raid0 
    └─ntlg-ntlg--lv       253:1    0  100M  0 lvm   /tmp/new
```

15. Протестируйте целостность файла:

    ```bash
    vagrant@sysadm-fs:/tmp/new$ gzip -t test.gz
	vagrant@sysadm-fs:/tmp/new$ echo $?
	0
    ```

16. Используя pvmove, переместите содержимое PV с RAID0 на RAID1.

```bash
root@sysadm-fs:/tmp/new# pvmove -n ntlg-lv /dev/md2 /dev/md1
  /dev/md2: Moved: 16.00%
  /dev/md2: Moved: 100.00%
```

17. Сделайте `--fail` на устройство в вашем RAID1 md.

```bash
root@sysadm-fs:/tmp/new# mdadm --fail /dev/md1 /dev/sdb1
mdadm: set /dev/sdb1 faulty in /dev/md1
```

18. Подтвердите выводом `dmesg`, что RAID1 работает в деградированном состоянии.

```bash
root@sysadm-fs:/tmp/new# dmesg | grep md1 | tail -n 2
[ 4115.760850] md/raid1:md1: Disk failure on sdb1, disabling device.
               md/raid1:md1: Operation continuing on 1 devices.
```

19. Протестируйте целостность файла — он должен быть доступен несмотря на «сбойный» диск:

	```bash    
	root@sysadm-fs:/tmp/new# gzip -t test.gz
	root@sysadm-fs:/tmp/new# echo $?
	0
	```
20. Погасите тестовый хост — `vagrant destroy`.

