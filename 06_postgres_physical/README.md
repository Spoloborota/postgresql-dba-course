# Установка и настройка PostgreSQL

## Цель:
- создавать дополнительный диск для уже существующей виртуальной машины, размечать его и делать на нем файловую систему
- переносить содержимое базы данных PostgreSQL на дополнительный диск
- переносить содержимое БД PostgreSQL между виртуальными машинами

В описании дз стоит уточнить, что в убунту 20.04 и 22.04 для остановки постгреса 15 работает команда:
> sudo systemctl stop postgresql@15-main  

а для запуска команда:
> sudo systemctl start postgresql@15-main  

В описании дз стоит уточнить, что после перезагрузки инстанса стоит вырубить кластер постгреса, если он автоматом поднялся  

В описании дз стоит подправить строку:
> /var/lib/postgres/14 в /mnt/data - mv /var/lib/postgresql/15/mnt/data   


на  
> предположим вы примонтировали диск в /mnt/data, тогда нужно переместить:  
/var/lib/postgresql/15 в /mnt/data - mv /var/lib/postgresql/15/* /mnt/data/   

т.к. в описании дз стоит версия 15, а в скрипте 14 и студент мог создать папку для монтирования с другим именем по инструкции в самом гуглоклауде - https://cloud.google.com/compute/docs/disks/add-persistent-disk   


После перемещения файлов сервис постгреса не стартанул потому что папки main не существует на прежнем месте:
> rustemsaifutdinov@instance-1:~$ systemctl status postgresql@15-main.service  
● postgresql@15-main.service - PostgreSQL Cluster 15-main  
Loaded: loaded (/lib/systemd/system/postgresql@.service; enabled-runtime; vendor preset: enabled)  
Active: failed (Result: protocol) since Sun 2023-03-19 22:14:17 UTC; 4min 2s ago  
Process: 1505 ExecStart=/usr/bin/pg_ctlcluster --skip-systemctl-redirect 15-main start (code=exited, status=1/FAILURE)  
>  
> Mar 19 22:14:17 instance-1 systemd[1]: Starting PostgreSQL Cluster 15-main...  
Mar 19 22:14:17 instance-1 postgresql@15-main[1505]: Error: /var/lib/postgresql/15/main is not accessible or does not exist  
Mar 19 22:14:17 instance-1 systemd[1]: postgresql@15-main.service: Can't open PID file /run/postgresql/15-main.pid (yet?) after start: Operation not permitted  
Mar 19 22:14:17 instance-1 systemd[1]: postgresql@15-main.service: Failed with result 'protocol'.  
Mar 19 22:14:17 instance-1 systemd[1]: Failed to start PostgreSQL Cluster 15-main.  

Поправил правкой строки в конфиге:
> data_directory = '/mnt/disks/new_disk/main'

После старта сервиса бд все записи по прежнему на месте:
> rustemsaifutdinov@instance-1:~$ sudo -u postgres psql  
psql (15.2 (Ubuntu 15.2-1.pgdg20.04+1))  
Type "help" for help.  
>   
> postgres=# SELECT * FROM test;  
c1  
`----`  
1  
(1 row)  

задание со звездочкой *:  
1. создал инстанс, запустил на нем постгрес 15:  
> sudo apt update && sudo apt upgrade -y && sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list' && wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add - && sudo apt-get update && sudo apt-get -y install postgresql-15  

> rustemsaifutdinov@instance-2:~$ systemctl status postgresql@15-main.service  
● postgresql@15-main.service - PostgreSQL Cluster 15-main  
Loaded: loaded (/lib/systemd/system/postgresql@.service; enabled-runtime; vendor preset: enabled)  
Active: active (running) since Sun 2023-03-19 22:47:46 UTC; 21s ago  
Process: 10140 ExecStart=/usr/bin/pg_ctlcluster --skip-systemctl-redirect 15-main start (code=exited, status=0/SUCCESS)  
Main PID: 10145 (postgres)  
Tasks: 6 (limit: 2356)  
Memory: 18.8M  
CPU: 173ms  
CGroup: /system.slice/system-postgresql.slice/postgresql@15-main.service  
├─10145 /usr/lib/postgresql/15/bin/postgres -D /var/lib/postgresql/15/main -c config_file=/etc/postgresql/15/main/postgresql.conf  
├─10146 "postgres: 15/main: checkpointer " "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "">  
├─10147 "postgres: 15/main: background writer " "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" >  
├─10149 "postgres: 15/main: walwriter " "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "">  
├─10150 "postgres: 15/main: autovacuum launcher " "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" ">  
└─10151 "postgres: 15/main: logical replication launcher " "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" ">  
>  
> Mar 19 22:47:43 instance-2 systemd[1]: Starting PostgreSQL Cluster 15-main...  
Mar 19 22:47:46 instance-2 systemd[1]: Started PostgreSQL Cluster 15-main.  

2. вырубил сервис постгреса на первом инстансе, отмонтировал диск:  
> rustemsaifutdinov@instance-1:~$ sudo systemctl stop postgresql@15-main  
rustemsaifutdinov@instance-1:~$ sudo umount /mnt/disks/new_disk  
rustemsaifutdinov@instance-1:~$ sudo lsblk  
NAME    MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT  
loop0     7:0    0  55.6M  1 loop /snap/core18/2697  
loop1     7:1    0  55.6M  1 loop /snap/core18/2714  
loop2     7:2    0  63.3M  1 loop /snap/core20/1822  
loop3     7:3    0  63.3M  1 loop /snap/core20/1828  
loop4     7:4    0 337.9M  1 loop /snap/google-cloud-cli/111  
loop5     7:5    0 339.5M  1 loop /snap/google-cloud-cli/115  
loop6     7:6    0  91.9M  1 loop /snap/lxd/24061  
loop7     7:7    0  49.8M  1 loop /snap/snapd/17950  
loop8     7:8    0  49.9M  1 loop /snap/snapd/18357  
sda       8:0    0    10G  0 disk  
├─sda1    8:1    0   9.9G  0 part /  
├─sda14   8:14   0     4M  0 part  
└─sda15   8:15   0   106M  0 part /boot/efi  
sdb       8:16   0    10G  0 disk  

3. отсоединил диск от первого инстанса, подключил ко второму:  
> rustemsaifutdinov@instance-1:~$ sudo lsblk  
NAME    MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT  
loop0     7:0    0  55.6M  1 loop /snap/core18/2697  
loop1     7:1    0  55.6M  1 loop /snap/core18/2714  
loop2     7:2    0  63.3M  1 loop /snap/core20/1822  
loop3     7:3    0  63.3M  1 loop /snap/core20/1828  
loop4     7:4    0 337.9M  1 loop /snap/google-cloud-cli/111  
loop5     7:5    0 339.5M  1 loop /snap/google-cloud-cli/115  
loop6     7:6    0  91.9M  1 loop /snap/lxd/24061  
loop7     7:7    0  49.8M  1 loop /snap/snapd/17950  
loop8     7:8    0  49.9M  1 loop /snap/snapd/18357  
sda       8:0    0    10G  0 disk  
├─sda1    8:1    0   9.9G  0 part /  
├─sda14   8:14   0     4M  0 part  
└─sda15   8:15   0   106M  0 part /boot/efi  

4. Подключил диск ко второму инстансу и подмонтировал его, без правки fstab, навесил прав:  
> rustemsaifutdinov@instance-2:~$ sudo lsblk  
NAME    MAJ:MIN RM   SIZE RO TYPE MOUNTPOINTS  
loop0     7:0    0  55.6M  1 loop /snap/core18/2697  
loop1     7:1    0  63.3M  1 loop /snap/core20/1822  
loop2     7:2    0 337.9M  1 loop /snap/google-cloud-cli/111  
loop3     7:3    0 111.9M  1 loop /snap/lxd/24322  
loop4     7:4    0  49.8M  1 loop /snap/snapd/18357  
sda       8:0    0    10G  0 disk  
├─sda1    8:1    0   9.9G  0 part /  
├─sda14   8:14   0     4M  0 part  
└─sda15   8:15   0   106M  0 part /boot/efi  
sdb       8:16   0    10G  0 disk  
rustemsaifutdinov@instance-2:~$ sudo mkdir -p /mnt/disks/new_disk  
rustemsaifutdinov@instance-2:~$ sudo mount -o discard,defaults /dev/sdb /mnt/disks/new_disk  
rustemsaifutdinov@instance-2:~$ sudo chmod a+w /mnt/disks/new_disk  
rustemsaifutdinov@instance-2:~$ sudo chown -R postgres:postgres /mnt/disks/new_disk/  

5. Подкорректировал конфиг постгреса и запустил постгрес:  
> rustemsaifutdinov@instance-2:~$ sudo nano /etc/postgresql/15/main/postgresql.conf  
rustemsaifutdinov@instance-2:~$ sudo systemctl start postgresql@15-main  

6. Нашел там все данные свои:
> rustemsaifutdinov@instance-2:~$ sudo -u postgres psql  
could not change directory to "/home/rustemsaifutdinov": Permission denied  
psql (15.2 (Ubuntu 15.2-1.pgdg22.04+1))  
Type "help" for help.  
>
> postgres=# \l  
List of databases  
Name    |  Owner   | Encoding | Collate |  Ctype  | ICU Locale | Locale Provider |   Access privileges  
-----------+----------+----------+---------+---------+------------+-----------------+-----------------------  
iso       | postgres | UTF8     | C.UTF-8 | C.UTF-8 |            | libc            |  
postgres  | postgres | UTF8     | C.UTF-8 | C.UTF-8 |            | libc            |  
template0 | postgres | UTF8     | C.UTF-8 | C.UTF-8 |            | libc            | =c/postgres          +  
|          |          |         |         |            |                 | postgres=CTc/postgres  
template1 | postgres | UTF8     | C.UTF-8 | C.UTF-8 |            | libc            | =c/postgres          +  
|          |          |         |         |            |                 | postgres=CTc/postgres  
(4 rows)  
>
> postgres=# SELECT * FROM test  
postgres-# \dt  
List of relations  
Schema | Name | Type  |  Owner  
--------+------+-------+----------  
public | test | table | postgres  
(1 row)  
> 
> postgres-# ;  
c1  
`----`  
1  
(1 row)  
>
> postgres=# SELECT * FROM test;  
c1  
`----`  
1  
(1 row)  