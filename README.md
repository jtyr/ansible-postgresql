postgresql
==========

Ansible role which helps to install and configure PostgreSQL server.

The configuration of the role is done in such way that it should not be
necessary to change the role for any kind of configuration. All can be
done either by changing role parameters or by declaring completely new
configuration as a variable. That makes this role absolutely
universal. See the examples below for more details.

Please report any issues or send PR.


Usage
-----

```
# Default usage with no configuration changes
- name: Example 1
  hosts: machine1
  roles:
    - postgresql

# Do some changes of the default server configuration (postgresql.conf file)
- name: Example 2
  hosts: machine2
  roles:
    - role: postgresql
      vars:
        # Change the default password for the postgres user
        postgresql_postgres_password: T0Ps3cr3t
        # Change the path for the data directory
        postgresql_data_directory: /mnt/pg/data
        # Change the server connection number
        postgresql_config_max_connections: 200

# Add custom server configuration
- name: Example 3
  hosts: machine3
  roles:
    - role: postgresql
      vars:
        postgresql_config__custom:
          # Add configuration to change the authentication timeout down to
          # 20 seconds
          authentication_timeout: 20s
          # Add configuration to increase the work memory to 10MB
          work_mem: 10MB

# Write the server configuration from scratch
- name: Example 4
  hosts: machine4
  roles:
    - role: postgresql
      vars:
        postgresql_config:
          authentication_timeout: 20s
          datestyle: iso, mdy
          default_text_search_config: pg_catalog.english
          lc_messages: en_US.UTF-8
          lc_monetary: en_US.UTF-8
          lc_numeric: en_US.UTF-8
          lc_time: en_US.UTF-8
          log_directory: pg_log
          log_filename: postgresql-%a.log
          logging_collector: 'on'
          log_rotation_age: 1d
          log_rotation_size: 0
          log_truncate_on_rotation: 'on'
          max_connections: 100
          shared_buffers: 32MB
          work_mem: 10MB

# Add IP from which we can connect to the server (pg_hba.conf file)
- name: Example 5
  hosts: machine5
  roles:
    - role: postgresql
      vars:
        # Allow access for the app1_user to the app1_db from 10.1.2.3
        postgresql_hba_config__custom:
          - type: host
            database: app1_db
            user: app1_user
            address: 10.1.2.3/32
            method: md5
        # Uncomment this to disable the by default configured local access
        #postgresql_hba_config__default: []

# Configure ident (system to db user linking - pg_ident.conf file)
- name: Example 6
  hosts: machine6
  roles:
    - role: postgresql
      vars:
        postgresql_ident_config:
          # Map some_system_user to some_db_user
          - mapname: some_user
            system: some_system_user
            pg: some_db_user

# Change the connection service settings (pg_service.conf file)
- name: Example 7
  hosts: machine7
  roles:
    - role: postgresql
      vars:
        postgresql_service_config:
          postgres:
            dbname: postgres
            user: postgres

# Change the recovery service setting (recovery.conf file)
- name: Example 8
  hosts: machine8
  roles:
    - role: postgresql
      vars:
        postgresql_recovery_config:
          recovery_target_time: 2004-07-14 22:39:00 EST
          recovery_target_xid: 1100842
          recovery_target_inclusive: 'true'
```

This role requires [Config
Encoders](https://github.com/jtyr/ansible/blob/jtyr-config_encoders/lib/ansible/plugins/filter/config_encoders.py)
which must be configured in the `ansible.cfg` file like this:

```
[defaults]

filter_plugins = ./plugins/filter/
```

Where the `./plugins/filter/` containes the `config_encoders.py` file.


Role variables
--------------

```
# Whether to install YUM repo
postgresql_yumrepo_install: yes

# YUM repo version
postgresql_yumrepo_version: 9.4

# YUM repo URL
postgresql_yumrepo_url: "{{ 'https://download.postgresql.org/pub/repos/yum/' + postgresql_yumrepo_version + '/redhat/rhel-$releasever-$basearch/' }}"

# Package to be installed (you can specify exact version here)
postgresql_pkg: postgresql-server

# Default DB password for the postgres user
postgresql_postgres_password: postgres

# Default data directory location
postgresql_data_directory: /var/lib/pgsql/data


# PostgreSQL HBA config file location
postgresql_hba_config_file: "{{ postgresql_data_directory }}/pg_hba.conf"

# Default PostgreSQL HBA config
postgresql_hba_config__default:
  - type: local
    database: all
    user: all
    method: trust
  - type: host
    database: all
    user: all
    address: 127.0.0.1
    netmask: 255.255.255.255
    method: trust
  - type: host
    database: all
    user: all
    address: ::1/128
    method: trust

# Custom PostgreSQL HBA config
postgresql_hba_config__custom: []

# Final PostgreSQL HBA config
postgresql_hba_config: "{{
    postgresql_hba_config__default +
    postgresql_hba_config__custom
  }}"


# PostgreSQL ident config file location
postgresql_ident_config_file: "{{ postgresql_data_directory }}/pg_ident.conf"

# PostgreSQL ident config
postgresql_ident_config: []
# Example:
#postgresql_ident_config:
#  - mapname: some_user
#    system: some_system_user
#    pg: some_db_user


# PostgreSQL service config file location
postgresql_service_config_file: "{{ postgresql_data_directory }}/pg_service.conf"

# PostgreSQL service config
postgresql_service_config: {}
# Example:
#postgresql_service_config:
#  postgres:
#    dbname: postgres
#    user: postgres


# PostgreSQL server config file location
postgresql_config_file: "{{ postgresql_data_directory }}/postgresql.conf"

# Default PostgreSQL server config options
postgresql_config_max_connections: 100
postgresql_config_shared_buffers: 32MB
postgresql_config_logging_collector: 'on'
postgresql_config_log_directory: pg_log
postgresql_config_log_filename: postgresql-%a.log
postgresql_config_log_truncate_on_rotation: 'on'
postgresql_config_log_rotation_age: 1d
postgresql_config_log_rotation_size: 0
postgresql_config_datestyle: iso, mdy
postgresql_config_lc_messages: en_US.UTF-8
postgresql_config_lc_monetary: en_US.UTF-8
postgresql_config_lc_numeric: en_US.UTF-8
postgresql_config_lc_time: en_US.UTF-8
postgresql_config_default_text_search_config: pg_catalog.english
postgresql_config_listen_addresses: 127.0.0.1

# Default PostgreSQL server config
postgresql_config__default:
  data_directory: "{{ postgresql_data_directory }}"
  max_connections: "{{ postgresql_config_max_connections }}"
  shared_buffers: "{{ postgresql_config_shared_buffers }}"
  logging_collector: "{{ postgresql_config_logging_collector }}"
  log_directory: "{{ postgresql_config_log_directory }}"
  log_filename: "{{ postgresql_config_log_filename }}"
  log_truncate_on_rotation: "{{ postgresql_config_log_truncate_on_rotation }}"
  log_rotation_age: "{{ postgresql_config_log_rotation_age }}"
  log_rotation_size: "{{ postgresql_config_log_rotation_size }}"
  datestyle: "{{ postgresql_config_datestyle }}"
  lc_messages: "{{ postgresql_config_lc_messages }}"
  lc_monetary: "{{ postgresql_config_lc_monetary }}"
  lc_numeric: "{{ postgresql_config_lc_numeric }}"
  lc_time: "{{ postgresql_config_lc_time }}"
  default_text_search_config: "{{ postgresql_config_default_text_search_config }}"
  listen_addresses: "{{ postgresql_config_listen_addresses }}"

# Custom PostgreSQL server config
postgresql_config__custom: {}

postgresql_config__tmp: {}

# Finale PostgreSQL server config
postgresql_config: "{{
  postgresql_config__tmp.update(
  postgresql_config__default) }}{{
  postgresql_config__tmp.update(
  postgresql_config__custom) }}{{
  postgresql_config__tmp }}"


# PostgreSQL recovery config file location
postgresql_recovery_config_file: "{{ postgresql_data_directory }}/recovery.conf"

# PostgreSQL recovery config
postgresql_recovery_config: {}
# Example:
#postgresql_recovery_config:
#  recovery_target_time: 2004-07-14 22:39:00 EST
#  recovery_target_xid: 1100842
#  recovery_target_inclusive: 'true'
```


Dependencies
------------

- [Config Encoders](https://github.com/jtyr/ansible/blob/jtyr-config_encoders/lib/ansible/plugins/filter/config_encoders.py)


License
-------

MIT


Author
------

Jiri Tyr
