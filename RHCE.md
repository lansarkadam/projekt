
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
  tasks:
    - name: Install Group list RPM
      ansible.builtin.yum:
        name: "@RPM Development Tools"
        state: present

- name: Install RPM
  hosts: dev

    - name: Update all packages
      ansible.builtin.yum:
        name: '*'
        state: latest
```


4. Create repo.yml for configuring repository in all hosts

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

`curl node1`

7. Cofigure roles.yml run balancer and phpinfo roles


UZUPEŁNIĆ


8. Install Ansible Collection


mkdir mycollection
cd mycollection

wget http:/// plik.tar.gz
wget http:/// plik2.tar.gz


ansible-galaxy collection install plik.tar.gz
ansible-galaxy collection install plik.tar.gz -p . force

ansible-galaxy collection install plik2.tar.gz
ansible-galaxy collection install plik2.tar.gz -p . force


9. Create a playbook webconent.yml and it should run on dev group


```
---
 - name:
   hosts: dev
   tasks:
     - name: create dir
       file:
         path: /devweb
         state: directory
         mode: '2775'
         group: devops
         setype: httpd_sys_content_t

     - name: create symbolic link
       file:
         src: /devweb
         dest: /var/www/html/devwqeb
         state: link
         setype: httpd_sys_content_t

      - name: copy using inline
        copy:
          content: "Development"
          dest: /devweb/index.html
          setype: httpd_sys_content_t

      - name: permit trafic
        firewalld:
          service: http
          state: enabled
          permanent: true
          immediate: true

```

`ansible-playbook webcontent.yml`

`curl node1/webdev/`

Wynik "Development"

10. Collect hardware report using playbook

 
`vim hwreport.yml`

```
---
- name: create hwreport
  hosts: all
  tasks:
    - name: download hwreport
      get_url:
        url: http://localfost/file/hwreport.empty
        dest: /root/hwreport.txt

```
11. Replace the file /etc/issue on all managed nodes

`vim issue.yml`

```
---
- name: content in dev
  hosts: dev
  tasks:
    - name: copy using inline content
      copy:
        content: Development
        dest: /etc/issue

- name: conetent in test
  hosts: test
  tasks:
    - name: copy using in line content
      copy:
        content: Test
        dest: /etc/issue

- name: content in prod
  hosts: prod
  tasks:
    - name: copy using in line content
      copy:
        content: Production
        dest: /etc/issue

```

`ansible all -m command -a 'cat /etc/issue'`

12. Download hosts.j2 and create /etc/myhosts

vim hosts.j2

```
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6

{% for host in griups['all']%}
{{ hostvars[host].ansible_default_ipv4.address }} {{ hostvars[host].ansible_fqdn }} {{hostvars[host].ansible_hostname }}
{% endfor %}
```

Plik już gotowy na egzaminie

`ansible-playbook host.yml`

13. Create vault.yml with encrypted variables

`vim secret.txt` - hasło podane na egzaminie

`ansible-vault create vault.yml --vault-password-file=secret.txt`

wpisanie zwartości

```
pw_developer : Iamdev
pw_manager : Immgr
```

`ansible-vault view vault.yml --vault-password-file=secret.txt` - podejrzenie
