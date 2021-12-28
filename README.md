# Ansible Role For SSH

Setup SSH secure configration with specific port.

# Examples

Directory tree.

```
.
├── README.md
├── ansible.cfg
├── authorized_keys
│   └── aintek
│        └── centos
│        └── ubuntu
├── group_vars
│   ├── all.yml
│   ├── webserver_centos.yml
│   └── webserver_ubuntu.yml
├── inventory
└── webservers.yml
```


## Inventory file

Bofore apply this role, use default ssh port `22`.

Note: You should fix ssh ports to `group_vars/all.yml`'s port number(`22222`) after this role applied.

```
[webservers:children]
webserver_ubuntu
webserver_centos

[webserver_ubuntu]
ubuntu ansible_host=10.10.10.10 ansible_port=22

[webserver_centos]
centos ansible_host=10.10.10.11 ansible_port=22
```

## Group Vars / Common settings(all.yml)

`all.yml` sets common variables.

This role refers `ssh` variable having `port` and
change the default port 22 to `ssh.port`

```
# Common settings
become: yes
ansible_user: root

# Private_key is saved in local host only!
ansible_ssh_private_key_file: ""

ssh:
  port: 22222
```

## Group Vars / Ubuntu(webserver_ubuntu.yml)

`webserver_ubuntu.yml` is `webservers` host's children.

```
ansible_user: ubuntu
become: yes
ansible_become_password: 'ThisIsSecret!'

users:
  - name: aintek
```

## Group Vars / CentOS(webserver_centos.yml)

`webserver_ubuntu.yml` is `webservers` host's children.

```
users:
  - name: aintek
```

## Authorized_keys

This role copy `authorized_keys` in `./authorized_keys/{{ user.name }}/{{ inventory_hostname }}`.

So when you want to apply this role to `ubuntu` and `centos` for `aintek` user(you want to create aintek's authorized_keys),
you should make `./authorized_kyes/aintek/ubuntu` and `./authorized_kyes/aintek/centos`.

# How to DryRun and Apply

DryRun

```
ansible-playbook -i inventory --private-key="~/.ssh/your_private_key" -CD webservers.yml --tags ssh
```

Apply

```
ansible-playbook -i inventory --private-key="~/.ssh/your_private_key" -D webservers.yml --tags ssh
```
