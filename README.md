# Резервное копирование - Александр Шевцов





![image](https://github.com/aztecprod/Reserve-Copy/assets/25949605/5e5a41fb-a0f5-4b76-9e18-15f7afe91d17)
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
