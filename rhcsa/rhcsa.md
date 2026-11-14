1. Zmiana nazwy hostname

```
hostnamectl set-hostname server2.example.localhost

bash
```


2. Konfiguracja repo

```
[BaseOS]
name = BaseOS
baseurl = https://cdn.redhat.com/content/dist/rhel8/$releasever/x86_64/baseos/os
enabled = 1
gpgcheck = 0

[AppStream]
name = AppStream
baseurl = https://cdn.redhat.com/content/dist/rhel8/$releasever/x86_64/baseos/os
enabled = 1
gpgcheck = 0
```

3. Zmiana portu dla httpd oraz dodanie do firewalld oraz konfiguracja semanage

- Zmienić port w poniższym pliku
```
vim /etc/httpd/conf/httpd.conf

systemctl restart httpd
```

- Dodanie port 82 do semanage
```
semanage port -l |grep http
man semanage-port

semanage port -a -t http_port_t -p tcp 82
```

- Dodanie portu 82 do firewalld

```
firewall-cmd --permanent --add-port=82/tcp
firewall-cmd --reload

systemctl restart httpd
```

```
ls -lZ /var/www/html/

chcon -t httpd_sys_content_t index.html
```

4. Dodanie grupy oraz użytkowników 

```
groupadd sysadmin
useradd -G sysadmin natasha
useradd -G sysadmin harry
useradd -s /sbin/nologin sarah
passwd natasha
passwd harry
passwd sarah
```
5. Create collaborative directory

```
mkdir -p /common/admin
ls -ld /common/admin/
chgrp sysadmin /common/admin/
chmod 2770 /common/admin/
```

6. Utworzenie cronjob

- ustawienie /bin/bash Hello o 14:23
```
crontab -eu natasha
23	14	*	*	*	/bin/echo Hello
```
- ustawienie logger-a co 3 minuty
```
crontab -eu natasha
*/3	*	*	*	*	/usr/bin/logger "EX200 in progress"
```

7. Wykonanie backupu katalogu /etc do lokalizacji /home za pomocą bzip2


```
yum install bzip2 -y
```
```
tar -cvjf /home/backup.tar.bz2 /etc
```

8. Konfiguracja NTP chrony

```
vim /etc/chrony.conf
```
wpisać

`server 2.rhel.pool.ntp.org iburst`

```
systemctl restart chronyd
chronyc sources -v
```

9. Znaleźć pliki użytkownika lub o jakies konkretnej wielkości i przekopiować do /root/found
```
mkdir -p /root/found
```

- użytkownika simeone

```
find / -user simeone -exec cp -arvf {} /root/found/ \;
```
- wielkośc plików 3M

```
find /etc -type f -size +4M -exec cp {} /files/largefiles/ \;
```

10. Find the string w plikach

```
grep "strato" /usr/share/dict/words > /searchfile
```

11. Configure autofs to automount the home directory of remoteuserx user. 

```
dnf install autofs nfs-utils -y
```

showmount -e adres IP lub nazwa na której wystawiony jest zasób

```
[root@server1 ~]# showmount -e 192.168.0.216
Export list for 192.168.0.216:
/software 192.168.0.215
```

```
vim /etc/auto.master.d/software.autofs
```
zawartość

```
/server2 /etc/auto.software --timeout=60
```
lub

```
/- /etc/auto.software --timeout=60
```

```
vim /etc/auto.software
```
zawartość
 - pierwsza opcja z "/server2"
```
automount -fstype=nfs4,rw,sync,initr 192.168.0.216:/software

```

lub

- druga opcja z "/-"
```
/automount -fstype=nfs4,rw,sync,initr 192.168.0.216:/software
```

12. Dodanie użytkownika o określonym UID

```
useradd -u 2112 barry
```

13. Copy /etc/fstab to /var/tmp oraz nadanie uprawnień przez setfacl

```
setfacl -m u:harry:rw /var/tmp/fstab
setfacl -m u:natasha:--- /var/tmp/fstab
getfacl /var/tmp/fstab
```

14. Config permision  - wykorzystanie umask

su - natasha

vim .bash_profile

dodać 

umask 0277

wylogować się z użytkownika i zalogować się ponownie

sprawdzić umask co zostało ustawione

15. Ustawienie zmiany hasła co 20 dni

vim /etc/login.defs
chage -l natasha
useradd test
chage -l test

16. Dodanie grupy admin do sudo without password

```
visudo
```
dodać

```
%admin ALL=(ALL)        NOPASSWD: ALL
```
17. Reset hasła root

wpisać na końcu

rd.break

mount -o remount,rw /sysroot/

chroot /sysroot/

passwd

podać nowe hasło dwa razy

touch /.autorelabel

18. SWAP dodanie nowej partycji 512M dla SWAP

fdisk /dev/sdb
n
p
+512M
t
82 - swap

w
q

```
mkswap /dev/sdb1
```

```
blkid
```

pobrać uid i wstawić do /etc/fstab

```
UUID="0437135a-6978-451e-992f-850e3eb28aec" swap 			  swap 	  defaults 	  0 0
```
```
swapon -a
swapon -s
```



19. Partycja LVM

 - 8 extend VOLUMEGROUP
 - 50 extend LOGICAL VOLUME GROUP


fdisk /dev/sdb

n
p

+2G

t
8e - lvm partition

w
q

 pvcreate /dev/sdb2

- utworzenie voluegroup 50 extendow
```
vgcreate -s 8MB datastore /dev/sdb2
vgs
vgscan
vgdisplay
```
- utworzenie logical volume 50 extendów - cała hartycja będzie miała wtedy 400MiB - taka informacja powinna być w zadaniu na egzaminie, ewentualnie trzeba pomnożyć itd.
```
lvcreate -l 50 -n database datastore
```

```
mkdir -p /mnt/database
blkid
mkfs.ext3 /dev/datastore/database
vim /etc/fstab
lvscan
mount -a
systemctl daemon-reload
```




