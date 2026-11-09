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

