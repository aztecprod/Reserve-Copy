# Резервное копирование - Александр Шевцов
![image](https://github.com/aztecprod/Reserve-Copy/assets/25949605/f0e4445a-7345-46af-8359-a64c3d8ba119)
1) При полном резервном копировании каждый раз делается копирование всех файлов, каталогов и т.д. 
2) При диф. копировании сначала делается полное копирование,  при каждом запуске процесса резервируются только измененные данные, но точкой отсчета является состояние времени полного бэкапа. 
3) Инкрементное копирование работает как дифференцированное копирование, но в отличии от него бэкапятся данные, которые были изменены из последнего слепка, то есть отправная точка каждого нового бэкапа это бэкап n-1.

![image](https://github.com/aztecprod/Reserve-Copy/assets/25949605/fd61c66d-311e-42d0-b604-1edba867f339)

Конфигурация bacula-sd.conf
```
Storage {                             # definition of myself
  Name = debian2-sd
  SDPort = 9103                  # Director's port
  WorkingDirectory = "/var/lib/bacula"
  Pid Directory = "/run/bacula"
  Plugin Directory = "/usr/lib/bacula"
  Maximum Concurrent Jobs = 20
  SDAddress = 127.0.0.1
}

Director {
  Name = debian2-dir
  Password = "12345"
}

Director {
  Name = debian2-mon
  Password = "12345"
  Monitor = yes
}

Autochanger {
Name = Autochanger1
Device = FileChgr1-Dev1, FileChgr1-Dev2
Changer Command = ""
Changer Device = /dev/null
}

Device {

Name = FileChgr1-Dev1

Media Type = File1

Archive Device = /backups/files1

LabelMedia = yes

Random Access = Yes

AutomaticMount = yes

RemovableMedia = no

AlwaysOpen = no

Maximum Concurrent Jobs = 1

}
Device {

Name = FileChgr1-Dev2

Media Type = File1

Archive Device = /backups/files2

LabelMedia = yes

Random Access = Yes

AutomaticMount = yes

RemovableMedia = no

AlwaysOpen = no

Maximum Concurrent Jobs = 1

}

Messages {

Name = Standard

director = dir-dir = all

}
```

Конфигурация bacula-fd.conf
```
Director {
  Name = debian2-dir
  Password = "12345"
}

Director {
  Name = debian2-mon
  Password = "12345"
  Monitor = yes
}

FileDaemon {                          # this is me
  Name = debian2-fd
  FDport = 9102                  # where we listen for the director
  WorkingDirectory = /var/lib/bacula
  Pid Directory = /run/bacula
  Maximum Concurrent Jobs = 20
  Plugin Directory = /usr/lib/bacula
  FDAddress = 127.0.0.1
}

# Send all messages except skipped files back to Director
Messages {
  Name = Standard
  director = debian2-dir = all, !skipped, !restored
}
```

Конфигурация bacula-dir.conf

```
Director {

Name = debian2-dir
DIRport = 9101
QueryFile = "/etc/bacula/scripts/query.sql"
WorkingDirectory = "/var/lib/bacula"
PidDirectory = "/run/bacula"
Maximum Concurrent Jobs = 1
Password = "12345"
Messages = Daemon
DirAddress = 127.0.0.1

}



Catalog {
Name = BaculaCatalog
DB Address = "127.0.0.1"
DB PORT = "5432"
dbname = "bacula"
dbuser = "bacula"
dbpassword = "12345"

}

FileSet {

Name = "Full Set"
Include {
Options {
signature = MD5
Compression = GZIP
aclsupport = yes
xattrsupport = yes

}

File = /var/lib/bacula/bacula.sql

}

}

Client {

Name = debian2-fd
Address = 127.0.0.1
FDPort = 9102
Catalog = BaculaCatalog
Password = "12345"
File Retention = 60 days
Job Retention = 6 months
AutoPrune = yes

}

Schedule {
Name = "WeeklyCycle"
Run = Full sun-sat at 23:10
}

JobDefs {
Name = "DefaultJob"
Type = Backup
Level = Incremental
Client = debian2-fd
FileSet = "Full Set"
Schedule = "WeeklyCycle"
Storage = debian2-sd
Messages = Standard
Pool = File
SpoolAttributes = yes
Priority = 10
Write Bootstrap = "/var/lib/bacula/%c.bsr"

}



Storage {
Name = debian2-sd
Address = 127.0.0.1
SDPort = 9103
Password = "12345"
Device = Autochanger1
Media Type = File1
Maximum Concurrent Jobs = 1
}



Pool {

Name = File
Pool Type = Backup
Recycle = yes
Volume Retention = 365 days
AutoPrune = yes
Maximum Volume Bytes = 50G
Maximum Volumes = 100
Label Format = "Vol-"

}


Messages {

Name = Standard
mailcommand = "/usr/sbin/bsmtp -h localhost -f \"\(Bacula\) \<%r\>\" -s \"Bacula: %t %e of %c %l\" %r"
operatorcommand = "/usr/sbin/bsmtp -h localhost -f \"\(Bacula\) \<%r\>\" -s \"Bacula: Intervention needed for %j\" %r"
mail = root = all, !skipped
operator = root = mount
console = all, !skipped, !saved
append = "/var/log/bacula/bacula.log" = all, !skipped
catalog = all

}


Messages {

Name = Daemon
mailcommand = "/usr/sbin/bsmtp -h localhost -f \"\(Bacula\) \<%r\>\" -s \"Bacula daemon message\" %r"
mail = root = all, !skipped
console = all, !skipped, !saved
append = "/var/log/bacula/bacula.log" = all, !skipped

}

Job {

Name = "BackupClient1"
JobDefs = "DefaultJob"

}
```
![image](https://github.com/aztecprod/Reserve-Copy/assets/25949605/b3aee853-0367-4eb2-949b-763a3aed4f50)
![image](https://github.com/aztecprod/Reserve-Copy/assets/25949605/c154ce4f-921b-4253-8105-08a47e44292b)


![image](https://github.com/aztecprod/Reserve-Copy/assets/25949605/5e5a41fb-a0f5-4b76-9e18-15f7afe91d17)

Конфигурация Сервера
```
pid file = /var/run/rsyncd.pid
log file = /var/log/rsyncd.log
transfer logging = true
hosts allow = 192.168.0.109
use chroot = false
#max verbosity = 5
munge symlinks = yes
[data]
path = /etc
uid = root
read only = yes
list = yes
comment = Data backup Dir
auth users = backup
secrets file = /etc/rsyncd.scrt
```
Скрипт клиента
```
#/bin/bash
date
syst_dir=/backup/
srv_name=debian2
srv_ip=192.168.0.108
srv_user=backup
srv_dir=data
echo "Start backup ${srv_name}"
mkdir -p ${syst_dir}${srv_name}/increment/
/usr/bin/rsync -avz  --progress --delete --password-file=/etc/rsyncd.scrt ${srv_user}@${srv_ip}::${srv_dir} ${syst_dir}${srv_name}/current/ --backup --backup-dir=${syst_dir}${srv_name}/increment/`date +%Y-%m-%d`/
/usr/bin/find ${syst_dir}${srv_name}/increment/ -maxdepth 1 -type d -mtime +30 -exec rm -rf {} \;
date
echo "Finish backup ${srv_name}"
```
После запуска скрипта:
![image](https://github.com/aztecprod/Reserve-Copy/assets/25949605/2a16a66d-8eed-4c93-b31b-c9b8f84cf129)

Результат резервного копирования:

![image](https://github.com/aztecprod/Reserve-Copy/assets/25949605/735e5fdc-755a-498a-959b-0ac2e90a50e9)
![image](https://github.com/aztecprod/Reserve-Copy/assets/25949605/2342dcfa-f280-46c7-bdb2-dbe9a0d72c5f)




