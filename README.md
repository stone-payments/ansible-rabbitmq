RabbitMQ
=========

Ansible role to install and configure RabbitMQ server.

## Note

A good practice is to have, before using this, clear knowledge of what are the plugins you're going to need. By
default, this role enables the management plugin, here is a example of a playbook that enables more plugins.

```yaml
---
- name: queue service
  hosts: your_hosts
  become: true
  roles:
    - role: stone-payments.rabbitmq
      rabbitmq_plugins:
        - rabbitmq_management
        - rabbitmq_shovel
        - rabbitmq_shovel_management
```

## Requirements
------------
Before using this role make sure that the machines that will compose the cluster connect to each other - via ping or
name resolution. Also make sure that they can connect to the Internet to download RabbitMQ package.

For RedHat machines make sure the machines are subscribed. Also, this role requires the previous installation of the
EPEL repositories. Alternatively, you can use our role
[stone-payments.epel](https://github.com/stone-payments/ansible-epel) to install EPEL.

## Role Variables
--------------

```yaml

# Take the package given by the OS/distrib
rabbitmq_os_package: false

# Allways install RabbitMQ, 
# unless it's already installed and you don't want it to be replaced
rabbitmq_install_enabled: true

# Install specific RabbitMQ version, unless it's redefined
rabbitmq_major: 3
rabbitmq_minor: 7
rabbitmq_patch: 4
rabbitmq_release: "*"
rabbitmq_version: "{{rabbitmq_major}}.{{rabbitmq_minor}}.{{rabbitmq_patch}}"

# TCP configuration
rabbitmq_conf_tcp_listeners_address: ''
rabbitmq_conf_tcp_listeners_port: 5672

# RabbitMQ configuration
rabbitmq_enabled_plugins_file_path: "/etc/rabbitmq/enabled_plugins"
rabbitmq_enabled_plugins_file_owner: root
rabbitmq_enabled_plugins_file_group: rabbitmq
rabbitmq_enabled_plugins_file_mode: 0644

rabbitmq_config_file_path: "/etc/rabbitmq/rabbitmq.config"
rabbitmq_env_variables_file_path: "/etc/rabbitmq/rabbitmq-env.config"
rabbitmq_config_file_owner: root
rabbitmq_config_file_group: rabbitmq
rabbitmq_config_file_mode: 0644

# Variables can be overriden to adapt to the user case
rabbitmq_conf_disk_free_limit_mem_relative: 1.5
rabbitmq_conf_vm_memory_high_watermark: 0.4 

# system number of open files
rabbitmq_service_d_path: /etc/systemd/system/rabbitmq-server.service.d
rabbitmq_system_number_open_files: 50000


# RabbitMQ vhosts
rabbitmq_vhosts:
  - anotherVhost

# RabbitMQ users
rabbitmq_users:
  - user: guest
    state: absent

  - user: admin
    password: adminpass
    permissions:
      - vhost: "/"
        configure_priv: .*
        read_priv: .*
        write_priv: .*
      - vhost: "{{ rabbitmq_vhosts[0] }}"
        configure_priv: .*
        read_priv: .*
        write_priv: .*
    tags: administrator

  - user: user2
    password: user2pass
    write_priv: .*
    vhost: "{{ rabbitmq_vhosts[0] }}"

# RabbitMQ plugins
rabbitmq_bin_path: "/usr/lib/rabbitmq/bin"
rabbitmq_sbin_path: "/usr/lib/rabbitmq/sbin"
rabbitmq_plugins_prefix_path: "/usr/lib/rabbitmq"
rabbitmq_plugins:
  - rabbitmq_management

# RabbitMQ cluster
rabbitmq_clustering_enabled: false
rabbitmq_erlang_cookie_file_path: "/var/lib/rabbitmq/.erlang.cookie"


# RabbitMQ repositories on satellite
rabbitmq_repository_on_satellite:
  - "repo-1"
  - "repo-2"

```

## Dependencies
------------

None yet.

## Example Playbook
----------------

```yaml
    - hosts: servers
      roles:
        - stone-payments.rabbitmq
```
By default this role will install the currently last upstream version of RabbitMQ (which is 3.7.15). If you want to install any other version you must give the version numbers.

```yaml
    - hosts: servers
      roles:
        - role: stone-payments.rabbitmq
          rabbitmq_major: 3
          rabbitmq_minor: 6
          rabbitmq_patch: 9
```

By default the role will try the configure the NODENAME with `rabbit@{{ansible_fqdn}}` when clustering is enabled, if you want to customize the node name, you can substitute the NODENAME with the variables `rabbitmq_nodename_prefix`and `rabbitmq_nodename_hostname`.

Others specific RabbitMQ environment variables can also be given.

```yaml
    vars:
      rabbitmq_conf_env:
        RABBITMQ_NODE_IP_ADDRESS: "127.0.0.2"
```

You can alter: 
* Memory watermark (`rabbitmq_conf_disk_free_limit_mem_relative`);
* Free disk space limit (`rabbitmq_conf_vm_memory_high_watermark`);
* Number of system's open files (`rabbitmq_system_number_open_files`). 

```yaml
    vars:
      rabbitmq_conf_disk_free_limit_mem_relative: 1.5
      rabbitmq_conf_vm_memory_high_watermark: 0.4
      rabbitmq_system_number_open_files: 50000
```

To create a cluster you just have to run this role against the target nodes and
give some extra vars.

```yaml
- name: queue service clustered
  hosts: group2
  become: true
  roles:
    - role: stone-payments.rabbitmq
      rabbitmq_erlang_cookie: <your_cookie>
      rabbitmq_clustering_enabled: true
      rabbitmq_master_node: "your_master_node"
```
To create a cluster using FQDN for hosts, just set USE_LONGNAME.

```yaml
    vars:
      rabbitmq_conf_env:
        USE_LONGNAME: "true"
```


Notice that the cookie is hash string that can be of any size. A good practice is
use a hash of 20 characters. This is the syncronization cookie used by erlang to
create the cluster.

Use this [playbook](playbook.yml) as a practical example.

## Testing
-------------

This role was developed using [Molecule](https://molecule.readthedocs.io). The `molecule.yml` and the `playbook.yml`
for testing are on the root of this role.

## License
-------

MIT

## To Do
-------------
  - Set specific permissions and priviledges to specific users
  - Add tests for the conection/read/write of rabbit's queues
  - Test removing flush_handlers from clustering step
  - Bug: when the master is down and the clustering step is run all the other nodes will go down.
  - App start with service start
