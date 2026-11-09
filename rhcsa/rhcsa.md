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


