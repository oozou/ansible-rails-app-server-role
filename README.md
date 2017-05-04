ansible-rails-app-server-role
=============================

A role to configure a debian based server to serve as a rails application
server.

WHAT DOES THIS ROLE DO EXACTLY?
-------------------------------

This role is comprised of many other roles (see DEPENDENCIES section). It setups
a debian based server to be a rails application server. It does this by doing
the following:

* basic debian based server setup
  * run [oozou.basic-server-setup](https://github.com/oozou/ansible-basic-server-setup) role
    * update apt
    * install essential packages
    * install other packages required by rails
      * nodejs
      * redis-server
      * redis-tools
      * imagemagick
      * libpq-dev
    * setup firewall using [oozou.ufw](https://github.com/oozou/ansible-ufw)
      * close all ports
      * open port 22
      * open port 80
      * open port 443
    * setup admin user account for devops user(s) using [oozou.add-admin-user](https://github.com/oozou/ansible-add-admin-user)
  * setup a user account for deploymment
    * setup ssh private and public keys
    * add specified ssh public keys to authorized_keys for deployment user
      * this can include circleci's ssh pub key for capistrano deployments
    * give sudo rights for restarting sidekiq and puma
  * install and setup ntp and nginx
    * run role [ansible-role-ntp](https://github.com/geerlingguy/ansible-role-ntp)
    * run role [ansible-role-nginx](https://github.com/jdauphant/ansible-role-nginx)
      * remove default site config
  * configure nginx
    * setup correct mime types
    * setup a vhost for your app with config options for [modern apps](https://certsimple.com/blog/nginx-http2-load-balancing-config).
  * install postgresql (if database.host is set to localhost)
    * install python-psycopg2
    * create database
    * create db user
  * deploy shared rails configuration files to server
    * interpolate templates and put config files on server in shared directory
      where capistrano symlinks the files into each code deployment


INSTALLATION
------------

Using ansible-galaxy:

```sh
$ ansible-galaxy install oozou.rails-app-server-role
```

Using requirements.yml:

```yaml
- src: oozou.rails-app-server-role
  path: roles/
  name: oozou.rails-app-server-role
```

ROLE VARIABLES
--------------

The following example is based on an ansible group\_vars file. Since this role
depends on the previously mentioned roles, it also relies on the vars required
by those roles. The group\_vars yaml below serves as a reference to a working
var file for completely configuring an Ubuntu 16.05 LTS server as a rails
application server running puma, nginx, and optionally postgresql.

**IT IS EXTREMELY IMPORTANT TO NOTE THAT THIS FILE CONTAINS SENSITIVE
INFORMATION THAT MAY BE USED TO COMPROMISE YOUR APP OR SYSTEM SECURITY!
YOU SHOULD NEVER COMMIT THIS FILE TO THE REPOSITORY IN PLAIN TEXT. USE
ansible-vault TO ENCRYPT THIS FILE PRIOR TO COMMITTING OR TRANSMITTING THIS
FILE ANYWHERE!**

```yaml
---
#
# Your application's name. The value here will be used as a name used to create
# directories, databases etc.
#
app_name: the_next_unicorn_app
basic_server_setup:
  additional_packages_to_install:
    - this_list_of_packages
    - will_be_joined_with
    - default_list_of_packages
    - in_the_following_role
    - oozou.basic-server-setup
#
# This is the user account for which you want to deploy the rails app as. This
# should be different from the admin account. Also configuration options for
# the deployer account like ssh keys, etc. There is also a configuration for
# all the public keys to put in authorized_keys file as well.
#
deployer_username: deploy
deployer:
  username: "{{deployer_username}}"
  home_dir: "/home/{{deployer_username}}"
  ssh_private_key: |
    -----BEGIN RSA PRIVATE KEY-----
    DEPLOYER_SSH_PRIVATE_KEY_MATCHING_PAIR_WITH_PUBLIC_KEY_BELOW_!!!
    h3yQiQ03YvnATd/glGUWxHc4LUAjL0Rnf3S9n+7Tk5kho9zsqMIjlBXcixwV83hQ
    SFWmEEPAEXeuRPwgqPDqatFih9/lhPChXKBlUprzaCtwOJCrlCyZ7o4tE74ZC0IN
    4E+4hLw+QqZmXzqwL2Q+GV0CSs8WS75QOI6rPM8rpQFJHeS0QY7D15mLSXmmFUvK
    jbjy9hWWyev+Tjzw0o1olG75PdSo8GOlRWndlbBIDzS5seEV+Lqa/wQBGcIfLoGK
    FDLKSJFLS  THIS IS JUST A SAMPLE!!!! DO NOT USE THIS KEY!!! )()S
    Cz3Wc1Z3gsL//vcsL5DfUTwnt9TZvcTCyi5zD1T5+U5SkqY1D99HW97QPN/b1I8o
    uZi3VjQ61RPhTGnyBoyLa9yQK5OLux2mlX678M0CgYA4Ue0mpH/iwWGv+jkynpVG
    4Ir/x1Xc66LrGIL1UL4JFfJBlXlf2qJUhYhGERf+esZggRusj1hNLlOAhpGoX180
    2LYNRsGGNs/yfgwNwnGkoQKBgCO0qcDvLLwf/uewQfhA25wp+4WikrPusMrf9WzB
    g5uZzHXg0xUmxDPgiMRCsTRToV2cJxsmaiHi0qkhPe2SuWajqxl1/M/kES3ZcM3+
    8flKwwpsk1y+tHvq+nIVavuHNmFRWwGbqSOTYPVZgY1M38WF+SNGosv3rCK/EVpm
    9yj9AoGAHrCsmE/vcrsJ4qMuZNIKz0j9XGcvP+wqvY9lNXM4Ff+Y3dP2VWYrCZB1
    t8eDpWgtIr5rMgJDu5rXFh0YSMu9UKb+WqWVZPYW/0oG8hTdmEBIOTUHKwpDOFGf
    SsUuQmI87B3kqfmnZDxmeztN0l2eh/Jheyw6aMwRS8ycZgn4Wns=
    -----END RSA PRIVATE KEY-----

  ssh_public_key: |
    ssh-rsa DEPLOYER_SSH_PUBLIC_KEY_AqcUumVDxoMH1D6ZWBMDxk5D/c9SM5/v8V/0cyELYUT+ynIPnO+nyVdU0R955xaCQqntOgZ3Q42N3tj9cYQM33ez0CkDn+TOTdujINJRunVrXEToCAFqmo7WHq0dpVd0QzuJD0DTz0lwWC6H6CljUwOqb9wt55fNbLNxG1qMbtiIOnjnSEsVdGGfhKuKkPIJdMrK1Hs7dC4wD9lovds982hi0o/h4goWg/gnw2GxkljDFlpAoCdgJmoaXsfv0SZDU3qX2nD28NQatOZQ/xOSzoeW4R9nRUTJi1q51HaIe9M1F3nMU3fwGS4We+76+fNv2CC67PeM8IW47RZosfRCuZJ6y1b9VKBvz3jPbFQfR4i0aPlsxIne6xVNp00MDU/4XK1l0D9vpWMdzxb/x0bFAJnS6rmWBDoWR5DQQXIqgr7KrBiyiRuV8hOzm7t6NdIFIlQWxMqwtBnV/XsVu86Lx6mVJgUmWcZgG02lPTDRbiLlSwrtTXaCd2+IBYD7e7S2/CHDE9eqmE9WNKxPgk2RIQ5miCEq+4g0QoFl6w== deployer@server.com

#
# These are the directories where your app will reside. No need to change these.
# These default to the deployer user's home directory/apps/app_name
#
app_root_dir: "{{deployer.home_dir}}/apps/{{app_name}}"
#
# The shared directory that capistrano expects all shared config files to reside
#
app_shared_dir: "{{app_root_dir}}/shared"
logical_deploy_environment: production
#
# rbenv section begin - install rbenv for the deployer user
#
rbenv_user: "{{deployer.username}}"
rbenv_ruby_version: 2.3.3
rbenv_users:
  - deploy
rbenv_gems:
  - bundler
# rbenv section end
#
# Database settings. If host is set to 'localhost' we will install postgresql
# on the app server
#
database:
  name: '{{app_name}}'
  password: SuperSecretPassword
  username: admin
  host: localhost
#
# Configuration for rails configuration files. The following assumes that you
# put templates in the templates directory which resides in the same directory
# as the playbook as specified by playbook_dir special ansible variable.
#
# The deploy_rails_config_dir var is expected to be the directory where the
# shared/linked config files used by capistrano reside on the server.
#
deploy_rails_env_vars:
  etc_environment:
    deploy: false
    dest: /etc/environment
  secrets_yml:
    deploy: true
    template: "{{playbook_dir}}/templates/secrets.yml.j2"
    dest: "{{deploy_rails_config_dir}}/secrets.yml"
  database_yml:
    deploy: true
    template: "{{playbook_dir}}/templates/secrets.yml.j2"
    dest: "{{deploy_rails_config_dir}}/database.yml"
  application_yml:
    deploy: true
    template: "{{playbook_dir}}/templates/application.yml.j2"
    dest: "{{deploy_rails_config_dir}}/application.yml"
  dot_rbenv_vars:
    deploy: true
    template: "{{playbook_dir}}/templates/dot.rbenv-vars.j2"
    dest: "{{deploy_rails_config_dir}}/dot.rbenv-vars"
    symlink_dest: "{{deployer.home_dir}}/.rbenv-vars"
    symlink_owner: "{{deployer.username}}"
    symlink_group: "{{deployer.username}}"
    create_symlink: true
  newrelic_yml:
    deploy: true
    template: "{{playbook_dir}}/templates/newrelic.yml.j2"
    dest: "{{deploy_rails_config_dir}}/newrelic.yml"
admin:
  email: user@example.com
  password: SuperSecretPassword123
rails:
  default_host: next.unicorn
  log_level: "debug"
  secret_key_base: "2641eea14b4 THIS IS NOT A REAL SECRET KEY BASE!!! e18a329e33f49df1c8d0bed5201cf5dcbda318724b70ac3dc461b3a4fd6d"
```

You should also create a `roles/users/vars/main.yml` file like so:

```yml
---
users:
  - description: circleci
    deployer: true
    ssh_key: |
      ssh-rsa AAAAxxxxxxxxxxxAAAA circle-ci
  - description: developer1
    deployer: true
    ssh_key: |
      ssh-rsa AAAAxxxxxxxxxxxAAAA developer@example.com
```

And include the file in your playbook.

EXAMPLE PLAYBOOK
----------------

```yml
---
- name: "Setup Rails App Server"
  hosts: rails
  vars_files:
    - roles/users/vars/main.yml
  become: true
  gather_facts: no

  #
  # Since Ubuntu 16.04 LTS does not come with python 2 installed by default,
  # you need to install it in order for ansible to work. The following lines
  # does this for you.
  #
  pre_tasks:
    - raw: sudo apt-get update
    - raw: test -e /usr/bin/python || sudo apt-get -y install python-simplejson
    - setup: # aka gather facts

  roles:
  - role: oozou.rails-app-server-role
    deployer_authorized_keys: "{{ users | selectattr('deployer','equalto',true) | map(attribute='ssh_key') | list }}"

```

INTENDED USAGE
--------------

You can create a devops directory under the root directory of your rails app
and create set of ansible playbooks there like so:

```
$ cd /path/to/your/rails/app/root/dir
$ cd devops
$ tree

├── README.md
├── hosts
│   └── production
│       ├── group_vars
│       │   └── rails
│       └── hosts
├── playbooks
│   ├── rails_server_setup.yml
│   ├── roles
|   |   └── users
|   |       └── vars
|   |           └── main.yml
│   └── templates
│       ├── application.yml.j2
│       ├── database.yml.j2
│       ├── dot.rbenv-vars.j2
│       ├── newrelic.yml.j2
│       └── secrets.yml.j2
└── requirements.yml

```

TAGS
------------

The individual tasks are tagged, so you can specify which task to run by passing in a tags parameter.

Available tasks:
  * basic_server_setup
  * deployer (use this if you just want to add/remove a user from authorized_keys of deployer)
  * ntp
  * nginx
  * systemd_services
  * rbenv
  * postgresql
  * deploy_rails_config (use this if you just want to update rails related config; e.g. application.yml)

Run an individual task like this:

```
ansible-playbook rails_server_setup.yml --tags 'deployer'
```

DEPENDENCIES
------------

* [oozou.basic-server-setup](https://github.com/oozou/ansible-basic-server-setup)
* [oozou.deploy-rails-env-vars](https://github.com/oozou/ansible-deploy-rails-env-vars)
* [oozou.ufw](https://github.com/oozou/ansible-ufw)
