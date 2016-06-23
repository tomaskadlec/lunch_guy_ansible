# Ansible - deploy an application

Tomáš Kadlec, 23. 6. 2016, Open@ICT

## Requirements

  * Install/Update runtime deps - nginx, php, php-fpm
  * Configure runtime deps
  * Install application and its dependencies
  * Configure application

## Environment

At first, virtual servers must be prepared. We will use a prepared servers
(from a template) running Debian 8 Jessie.

### Step 1: Create servers using OpenNebula SunStone UI

Click, click and click once again.

### Step 2: Create an inventory file.

Create ``inventory/hosts``, add generic configuration for all hosts.

```
[all:vars]
ansible_user=root
ansible_ssh_user=root
ansible_python_interpreter=/usr/bin/python2.7
```

Add hosts that will be managed.

```
web ansible_ssh_host=onevm-120.servers.ict.fit.cvut.cz

```
Try to ping added hosts.

```
kadleto2@host ansible-160623 $ ansible web -m ping
web | success >> {
    "changed": false,
    "ping": "pong"
}

```

## Install required runtime software

Use [apt module](http://docs.ansible.com/ansible/apt_module.html). Store list of
required software in a variable. Variables may be stored in *inventory* but they
are stored in dedicated files typically. These files are 

  * ``group_vars/`` - valid for a whole group of hosts, file is named by the group;
  * ``host_vars/`` - valid for a single host, file is named by the host.

Required software packages are:

```
php5, php5-imagick, php5-json, php5-fpm, php5-gd, php5-intl, php5-mcrypt, php5-mysqlnd, php5-readline, php5-tidy, php5-xsl, nginx
```

