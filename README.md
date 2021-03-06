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

## Configure software

Ansible provides modules that simplify inplace configuration changes as well as
synchronization of configuration files.

### Step 1: PHP configuration

Changes in configuration are necessary. Application will fail otherwise. Use 
[lineinfile](http://docs.ansible.com/ansible/lineinfile_module.html) module.

  * ``date.timezone`` -> Europe/Prague
  * ``memory_limit`` -> 256M
  * ``max_execution_time`` -> 120 (seconds)

``php-fpm`` must be restarted if configuration changes. Use a handler.

### Step 2: nginx configuration

Configuration file must be provided if no such exists. Allow its reuse using
[template](http://docs.ansible.com/ansible/template_module.html) module. File 
should be located at ``templates/nginx_app.conf``.

## Deploy application

Deploy [Obedar]() a PHP application for retrieving lunch menus in Dejvice, Prague, 
The Czech Republic. Following steps must be taken:

  * checkout source code from git VCS,
  * prepare composer and run ``composer install``,
  * set as current version if everything goes well.

### Step 0 - directory layout

Prepare directory layout similar to Capistrano/Capifony as depicted below.

```
.
|-- current -> /var/www/obedar/releases/__RELEASE__
|-- releases
|   `-- __RELEASE__
|       |-- app
|       |-- bin
|       |-- src
|       |-- var
|       |-- vendor
|       `-- web
`-- shared
```

### Step 1 - git checkout

Use [git]() module to checkout the application (master branch). Store it
in ``{{app.dest}}``.

### Step 2 - composer install

At first, [composer](http://getcomposer.org) must be installed, executable
and in ``$PATH``. Install it in ``/usr/local/bin``.

Run ``install`` command using [composer module](http://docs.ansible.com/ansible/composer_module.html).

```
- name: "app - composer install"
  composer:
    working_dir: "{{app.dest}}"
    no_dev: no
    command: install
```

Change ownership of application files, use ``www-data:www-data`` which are used for *php-fpm* and *nginx*. Use [file module](http://docs.ansible.com/ansible/file_module.html)

As the last step, change ``current`` symlink to the new release.
