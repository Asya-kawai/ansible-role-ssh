# Ansible Role For SSH

[![CI](https://github.com/Asya-kawai/ansible-role-ssh/actions/workflows/ci.yml/badge.svg)](https://github.com/Asya-kawai/ansible-role-ssh/actions/workflows?query=workflow%3ACI)

Setup SSH secure configration with specific port.

Caution: Configure `sshd_config` to allow `PubkeyAuthentication` only(disale passowrd authentication).
If you don't set authorized_keys, you can't login servers after applied this role perhaps.

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

Note: You should fix ssh ports to `group_vars/all.yml`'s port number(ex.`22222`) after this role applied.

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
change the default port 22 to `ssh_port`

```
# Common settings
become: yes
ansible_user: root

# Private_key is saved in local host only!
ansible_ssh_private_key_file: ""

ssh_port: 22222
```

## Group Vars / Ubuntu(webserver_ubuntu.yml)

`webserver_ubuntu.yml` is `webservers` host's children.

This role refers `users` variable having `name`,`home_dir` and `authorized_keys_file`.

```
ansible_user: ubuntu
become: yes
ansible_become_password: 'ThisIsSecret!'

users:
  - name: aintek
    home_dir: /home/aintek
    authorized_keys_file: '/path/to/your_home/.ssh/authorized_keys'
```

## Group Vars / CentOS(webserver_centos.yml)

`webserver_ubuntu.yml` is `webservers` host's children.

```
users:
  - name: aintek
    home_dir: /home/aintek
    authorized_keys_file: '/home/aintek/.ssh/authorized_keys'
```

## Group Vars CAUTION!

When you change ssh_port and enable firewall,
ansible may disconnect to the server.

As a result, you can not connect to the server on ssh_port,
until restart the server.

In particular, RHEL does not fully function firewall tasks...
Even after reboot, you will not be able to login(it's a bug).

If you want to enable firewall,
select an alternate role for firewall.
ex) [ansible-role-firewall](https://github.com/Asya-kawai/ansible-role-firewall)

```
# We don't recommend 'true'.
ssh_firewall_enable: false
```

# How to DryRun and Apply

DryRun

```
ansible-playbook -i inventory --private-key="~/.ssh/your_private_key" -CD webservers.yml --tags ssh

ansible-playbook -i inventory --private-key="~/.ssh/your_private_key" -CD webservers.yml --tags ssh-ufw
```

Apply

```
ansible-playbook -i inventory --private-key="~/.ssh/your_private_key" -D webservers.yml --tags ssh

ansible-playbook -i inventory --private-key="~/.ssh/your_private_key" -D webservers.yml --tags ssh-firewalld
```
