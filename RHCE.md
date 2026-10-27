
git add plik.yml
git commit -m "task1" plik.yml


1. Install and configure ansible

sudo yum install ansible-core ansible-navigator -yum

mkdir -p roles/
ansible-config init --disabled > ansible.config
lub
ansible-config init --disabled > ansible-config-disabled.yml.
ansible.cfg

[defaults]
inventory = /home/uzytkownik/ansible/inventory
remote_user = uzytkownik
roles_path = 
colection_path = 

[privilege_escalation]
become = True


vim inventory

3. Install Software Packages

RPM Developmnet, mariadb, php

ansible-navigator run packages.yaml -m stdout

```
---
- name: Install Packages
  hosts: dev,test,prod
  tasks:
    - name: Install MariaDb i PHP
      ansible.builtin.yum:
        name: "{{ packages }}"
      vars:
        packages:
        - mariadb
        - php
      state: present

- name: Install RPM
  hosts: dev
  tasks:
    - name: Install Group list RPM
      ansible.builtin.yum:
        name: "@RPM Development Tools"
        state: present
    - name: Update all packages
      ansible.builtin.yum:
        name: '*'
        state: latest
```


45. Create repo.yml for configuring repository in all hosts

```
---
- name: config repo
  hosts: all
  tasks:
    - name: Import a key from url
      rpm_key:
        key: url do klucza GPG

    - name: add baseos
      yum_repository:
        name: baseos-internal
        description: BaseOs Description
        file: baseos-internal
        baseurl: link do repozytorium base os
        gpgcheck: yes
        gpgkey: url do klucza GPG
        enabled: yes

    - name: add appstream
      yum_repository:
        name: appstream-internal
        description: App Description
        file: appstream-internal
        baseurl: link do repozytorium appstream
        gpgcheck: yes
        gpgkey: url do klucza GPG
        enabled: yes
```

5. Create directory 'roles' under /home/ansible/student and download roles

Role:
 - balancer.tar.gz
- phpinfo.tar.gz


`mkdir roles`
`cd roles`


```
---
- src: http://localhost/files/balancer.tar.gz
  name: balancer
- src: http://localhost/files/phpinfo.tar.gz
  name: phpinfo
```

`ansible-galaxy -r requirment.yml -p .`

6. Create offline role named apache under roles directory

`cd roles/`

`ansible-galaxy init apache`

```vim apache/templates/template.j2```

Zawartość: 

```
Welcome to {{ ansible_fqdn }} on {{ ansible_default_ipv4_address }}
```


`vim apache/tasks/main.yml`

```
---
# tasks file for apache
 - name: Install apache
   dnf:
     name: httpd
     state: latest

 - name: Start service httpd
   service:
     name: httpd
     state: started
     enabled: yes

 - name: start service firewalld
   service:
     name: firewalld
     state: restarted
     enable: yes

 - name: add http service in firewalld
   firewalld:
     service: http
     state: enabled
     permanent: yes
     immediate: yes

 - name: copy the template.j2
   template:
     src: template.j2
     dest: /var/www/html/index.html

```

cd /home/student/ansible/

vim apache_role.yml

```
---
- name: use apache role
  hosts: dev
  roles:
    - apache
```

`ansible-playbook apache_role.yml`

