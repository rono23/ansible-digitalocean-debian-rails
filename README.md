This is an Ansible playbook to setup Nginx, PostgreSQL, Unicorn etc for Ruby on Rails on Debian in DigitalOcean.

Please be aware this playbook is not idempotent.

## Setup
[![Gitter](https://badges.gitter.im/Join Chat.svg)](https://gitter.im/rono23/ansible-digitalocean-debian-rails?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge&utm_content=badge)

```
$ sudo easy_install pip
$ sudo pip install paramiko PyYAML jinja2 httplib2
$ brew install ansible
```

## Edit variables

```
# vars.yml
digital_ocean:
  client_id: "xxxxxxxxxx"
  api_key: "xxxxxxxxxx"

# vars/staging.yml
ssh:
  pub_key: "{{ lookup('file', '~/.ssh/id_rsa.pub') }}"
  deploy_user_private_key: "~/.ssh/{{ app }}_{{ user.name }}" # default is "example_deployer" without password.
```

## Run

All passwords (user, database etc) are 'password'.

```
$ ansible-playbook setup.yml -i localhost -K --extra-vars env=staging
$ [deploy your rails app]
```

After you created a droplet:
```
# only run some tasks: http://docs.ansible.com/playbooks_tags.html
$ ansible-playbook setup.yml -i staging -K --extra-vars env=staging -t nginx

# new playbook
$ ansible-playbook new-playbook.yml -i staging --extra-vars env=staging
```

## Security

This playbook saved raw passwords, but it shouldn't.
Use [ansible-vault](http://docs.ansible.com/playbooks_vault.html), [lookup](http://docs.ansible.com/playbooks_lookups.html) etc.

## Others

##### Create a password using SHA512 for root, user etc.

```
$ gem install unix-crypt
$ ruby -e "require 'unix_crypt'; puts UnixCrypt::SHA512.build('password')"
```

##### Get id to setup a droplet

- image_id: https://api.digitalocean.com/images/?client_id=xxx&api_key=xxx
- region_id: https://api.digitalocean.com/regions/?client_id=xxx&api_key=xxx
- size_id: https://api.digitalocean.com/sizes/?client_id=xxx&api_key=xxx
- ssh key ids: https://api.digitalocean.com/ssh_keys/?client_id=xxx&api_key=xxx

##### When you use Capistrano, check your deploy.rb:

```
set :linked_files, %w{config/database.yml config/secrets.yml config/unicorn.rb}
set :linked_dirs, %w{bin log tmp/pids tmp/cache tmp/sockets vendor/bundle public/system}
set :unicorn_config_path, 'config/unicorn.rb'
```


##### Warnings

- When you get ```msg: SSH Key failed to be created```, you can't have two SSH keys with the same contents under the same account.
