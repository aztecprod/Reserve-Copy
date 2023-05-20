# Резервное копирование - Александр Шевцов
![image](https://github.com/aztecprod/Reserve-Copy/assets/25949605/f0e4445a-7345-46af-8359-a64c3d8ba119)
1) При полном резервном копировании каждый раз делается копирование всех файлов, каталогов и т.д. 
2) При диф. копировании сначала делается полное копирование,  при каждом запуске процесса резервируются только измененные данные, но точкой отсчета является состояние времени полного бэкапа. 
3)Инкрементное копирование работает как дифференцированное копирование, но в отличии от него бэкапятся данные, которые были изменены из последнего слепка, то есть отправная точка каждого нового бэкапа это бэкап n-1.





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




