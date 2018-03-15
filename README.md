proxysql
========

A ProxySQL role capable proxying secure connections to a MySQL group replication cluster.

Requirements
------------

Requires a MySQL group configured by DigitalOcean's MySQL role (to grab the SSL assets necessary to secure the connection to the backend group).

Role Variables
--------------

```
# Type: boolean
# Default: false
# Description: Whether the backends are using multi-primary replication or not.
proxysql_multi_primary: true

# Type: string
# Default: /var/lib/proxysql
# Description: The directory to store ProxySQL's data files
proxysql_datadir: /var/lib/proxysql

# Type: list
# Default: (none)
# Description: A list of MySQL servers configured with group replication to use
# as the backend.  It's probably easiest to populate this with an Ansible
# group, as shown here.
proxysql_mysql_backend_servers: "{{ groups['mysql_nodes'] }}"

# Type: list of dictionaries
# Default: (none)
# Description: A list of dictionaries defining MySQL users that should be
# accessible.  These groupings will be used to create the ProxySQL accounts to
# pass through, and will be used to create local MySQL option files to ease
# logging in.  Keys that should be defined for each item are: `user`,
# `password`, `filename`, `port`, and `prompt`.  An admin user for the local
# ProxySQL instance (connecting on proxy_admin_interface` and
# `proxy_admin_port`) is always required.
proxysql_user_data:
    - user: "{{ proxysql_admin_user }}"
      password: "{{ proxysql_admin_password }}"
      filename: .my.cnf
      port: "{{ proxysql_admin_port }}"
      prompt: "ProxySQLAdmin> "
    - user: testuser
      password: testpass
      filename: testuser.cnf"
      port: "{{ proxysql_mysql_port }}"
      default_hostgroup: 2
      prompt: "{{ proxysql_group_test_user }}> "

# Type: boolean
# Default: false
# Description: Whether SSL is required by the MySQL backend group.
proxysql_require_ssl: true

# Type: boolean
# Default: false
# Description: Whether ProxySQL should use private network to connect to the backend hosts
proxysql_private_networking: true

# Type: integer
# Default: 1
# Description: The identifier for the group replication "offline" hostgroup.
proxysql_GR_hostgroups_offline_hostgroup: 1

# Type: integer
# Default: 2
# Description: The identifier for the group replication "writer" hostgroup.
proxysql_GR_hostgroups_writer_hostgroup: 2

# Type: integer
# Default: 3
# Description: The identifier for the group replication "reader" hostgroup.
proxysql_GR_hostgroups_reader_hostgroup: 3

# Type: integer
# Default: 4
# Description: The identifier for the group replication "backup writer"
# hostgroup.
proxysql_GR_hostgroups_backup_writer_hostgroup: 4

# Type: integer
# Default: 1 when multi-primary is off, # of backends when multi-primary is on
# Description: The maximum number of writers that should be active in a hostgroup.
proxysql_GR_hostgroups_max_writers: "{{ proxysql_mysql_backend_servers | length if proxysql_multi_primary else 1 }}"

# Type: integer (as boolean)
# Default: 1 (indicating true)
# Description: ProxySQL uses integers to represent some booleans.  This
# variable indicates whether hosts in the writer hostgroup should also be
# automatically added to the reader hostgroup.
proxysql_GR_hostgroups_writer_is_also_reader: 1

# Type: integer (as boolean)
# Default: 1 (indicating true)
# Description: ProxySQL uses integers to represent some booleans.  This
# variable indicates whether the hostgroup should be marked as active
# immediately.
proxysql_GR_hostgroups_active: 1

# Type: integer
# Default: 100
# Description: The maximum number of transactions a backend host can be behind before it is shunned by the ProxySQL host (taken out of rotation).
proxysql_GR_hostgroups_max_transactions_behind: 100
```

There are many additional variables that are parameterized versions of the ProxySQL admin and mysql variables.  You can find more about what each is responsible for by reading the [ProxySQL wiki page on global variables](https://github.com/sysown/proxysql/wiki/Global-variables).

Check this role's `defaults/main` file for a full list.  Admin variables use the prefix `proxysql_admin_` and mysql variables use the prefix `proxysql_mysql_`.  While most of the defaults are probably fine, you should change the variables associated with credentials (except for `proxysql_admin_cluster_username` and `proxysql_admin_cluster_password`) before deployment:

* proxysql_admin_user
* proxysql_admin_password
* proxysql_admin_stats_user
* proxysql_admin_stats_password
* proxysql_mysql_monitor_username
* proxysql_mysql_monitor_password

Additionally, there are some container variables configured in this role's `vars/main.yml` file that interpolate and structure the user editable variables.  These are internal structures that you will not need to edit yourself.

Dependencies
------------

* DigitalOcean's `mysql` role for the backend MySQL servers.

Example Playbook
----------------

```
---
- name: Set up ProxySQL to balance group
  hosts: mysql_proxy
  vars:
    proxysql_admin_user: admin
    proxysql_admin_password: admin
    proxysql_mysql_backend_servers:
      - mysql-node-1
      - mysql-node-2
      - mysql-node-3
    proxysql_require_ssl: true
    proxysql_group_monitor_user: monitor
    proxysql_group_monitor_pass: monitorpassword
    proxysql_user_data:
        - user: "{{ proxysql_admin_user }}"
          password: "{{ proxysql_admin_password }}"
          filename: .my.cnf
          port: "{{ proxysql_admin_port }}"
          prompt: "ProxySQLAdmin> "
        - user: testuser
          password: testpass
          filename: "testuser.cnf"
          port: "{{ proxysql_mysql_port }}"
          default_hostgroup: 2
          prompt: "testuser> "

  roles:
    - role: proxysql
```

License
-------

MIT

Author Information
------------------

DigitalOcean Community
