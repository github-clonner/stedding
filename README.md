# Stedding
Stedding is a minimalistic LEMP Stack setup for [Laravel PHP](https://laravel.com/) *in progress*. It facilitates the setting up of Laravel apps on a well prepared server using Ansible Playbooks on a [Ubuntu](https://www.ubuntu.com/) server. It provisions your VPS with all the tools necessary to run your Laravel PHP application with ease.

## Note
Again, this is still a work in progress. So use it wisely and backup when possible. Better still to test on a barebone server and send in pull requests :)

## Local Box Requirements
You need to have Ansible installed on your local computer. This really differs from box to box See [Ansible Documents](http://docs.ansible.com/ansible/intro_installation.html) for instructions. 

## Remote Server Requirements
To run Ansible Playbooks properly on *Ubuntu 16.0.4* we need to setup a sudo user and make sure Python and some other packages such as `ppa:ondrej/php` are available so Ansible can run. The setting up of a sudo user and adding of the SSH keys has been taken care of. So is the adding of Python and Ondrej's PHP PPA. All you need is root access to the Ubuntu 16.0.4 box. Preferably using an SSH key.

**NB** [Gist with useful setup tips](https://gist.github.com/jasperf/0be4439bbda9a324dd24e7300f357eb4)

## Playbooks

* Install prerequisites
* Sudo user Creation
* Web user Creation
* LEMP Provisioning
* Laravel Homebase Setup

## Roles

* [geerlingguy.nginx](https://github.com/geerlingguy/ansible-role-nginx)
* [geerlingguy.php](https://github.com/geerlingguy/ansible-role-php)
* [geerlingguy.mysql](https://github.com/geerlingguy/ansible-role-mysql)
* [geerlingguy.php-mysql](https://github.com/geerlingguy/ansible-role-php-mysql)
* [geerlingguy.memcached](https://github.com/geerlingguy/ansible-role-memcached)
* [geerlingguy.git](https://github.com/geerlingguy/ansible-role-git)
* [geerlingguy.composer](https://github.com/geerlingguy/ansible-role-composer)
* [Ansible Deployer](https://github.com/jverdeyen/ansible-deployer-in) - not in use as of yet
* [Ansible Users](https://github.com/singleplatform-eng/ansible-users) - not in use as of yet

## Stedding Variables
Do not forget to adjust the vars in 

* `grousp_var/all` and 
* `vars/mainyml` 

where need be. Not all will have to be adjusted perhaps but some will have to. This is besides the addition of the hosts file as will be mentioned later on. The variables in `vars/main.yml` are for setting up PHP, MySQL and Nginx details based on Geerlingguy roles. The variables in `grousp_var/all` are for the user only at the moment. Might merge all variables some time soon.

## Local Ansible Config Setup

We expect you to have installed Ansible on your own control box already. If not check out Ansible for [instructions](http://docs.ansible.com/ansible/intro_installation.html). 
### Adding Host to Hosts file
The Ansible config file is in the repository already. It checks for a *hosts* file the root of the project. It is put on .gitignore as we do not want to share host details. So you need to add it.
So create and open hosts file with nano using:
````
nano hosts
````
add php details using your non sudo user, laravel here, and the ip address to your server
````
[server]
xxx.xxx.xx.xxx
````
### Ansible Books Test
To do a test from your local computer - a MacBook Pro for example - you should run the following command:
````
ansible server -m ping
````
And when all is well you should get this response:
````
xxx.xxx.xx.xxx | SUCCESS => {
    "changed": false, 
    "ping": "pong"
}
````

### Run Playbook
Then to run the script use the following:
````
ansible-playbook server.yml
````
This is run as root in most of our cases `--ask-sudo-pass` is not added here.
## Server Packages
The current Ansible playbooks contain all the necessary packages to run a Laravel app and some more besides that. Here are all the packages:

* git
* nginx
* composer
* php 7.1
* mariadb
* memcached

#### Nginx
Nginx details are stored in `vars/main.yml` . One host for the site being used for testing purposes has been added there. Do change it to work with the domain of your choice.

````
nginx_remove_default_vhost: true
nginx_vhosts:
  - listen: "80 default_server"
    server_name: "{{domain}}"
    root: "/var/www/{{domain}}/public"
    index: "index.php index.html index.htm"
    state: "present"
    template: "{{ nginx_vhost_template }}"
    extra_parameters: |
      location / {
          try_files $uri $uri/ /index.php$is_args$args;
      }

      location ~ \.php$ {
          fastcgi_split_path_info ^(.+\.php)(/.+)$;
          fastcgi_pass unix:/var/run/php/php7.1-fpm.sock;
          fastcgi_index index.php;
          fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
          include fastcgi_params;
      }
````

#### PHP

Current PHP configuration details added to `vars/main.yml` are:
````
php_memory_limit: "512M"
php_max_execution_time: "90"
php_upload_max_filesize: "256M"
php_version: "7.1"
php_packages_state: latest
php_packages:
  - php7.1-apcu
  - php7.1-common
  - php7.1-intl
  - php7.1-cli
  - php7.1-dev
  - php7.1-fpm
  - libpcre3-dev
  - php7.1-gd
  - php7.1-curl
  - php7.1-imap
  - php7.1-json
  - php-mbstring
  - php7.1-mcrypt
  - php7.1-opcache
  - php7.1-pdo
  - php7.1-xml
  - php7.1-mbstring
  - php7.1-zip
  - php7.1-mysql
php_date_timezone: "Europe/Amsterdam"
php_webserver_daemon: "nginx"
php_fpm_daemon: php7.1-fpm
````

To work with PHP 7.1. Ondrej's PHP PPA is added in requirements playbook using:
````
- name: Add repository for PHP 7.
      apt_repository: repo='ppa:ondrej/php'
````

And to make sure all the Ubuntu PHP related config files get all the settings we have to add:
````
php_conf_paths:
  - /etc/php/7.1/fpm
  - /etc/php/7.1/cli

php_extension_conf_paths:
  - /etc/php/7.1/fpm/conf.d
  - /etc/php/7.1/cli/conf.d
````
##### PHP Packages
Current list of PHP packages as listed above is pretty large at the moment and not all are needed to run Laravel. In the future some of these packages may be removed.

#### MySQL

The MySQL details added to `vars/main.yml` so far are only for adding a dummy database:
````
mysql_root_password: super-secure-password
mysql_databases:
  - name: example_db
    encoding: latin1
    collation: latin1_general_ci
mysql_users:
  - name: example_user
    host: "%"
    password: similarly-secure-password
    priv: "example_db.*:ALL"
````

More details will most probably be added at a later stage.

### Laravel Homebase Setup

To run your Laravel application from a specific project directory, the one added to your Nginx configuration, we have added a separate playbook. One we will expand upon soon with other tasks. For now the project directory is created only using this task:
```
  - name: Project Folder Creation
    file: dest=/var/www/{{domain}} mode=2755 state=directory owner=web group=www-data
````
The domain can be set in `group_vars/all`. [GUID](https://blog.dbrgn.ch/2014/6/17/setting-setuid-setgid-bit-with-ansible/) has been set as well so all files and directories added will all be under group www-data. User web should be used to add files in the project folder preferably as it is the owner of the project directory.

### Deployment
Deployment script using [Deployer.org](https://deployer.org/) has been added as a role to this Ansible package. It is using the latest role version that is available on Github. The actual command to install the Laravel necessary files by Deployer:
````
dep init -t Laravel
````
has not been added as of yet.