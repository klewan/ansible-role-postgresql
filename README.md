Ansible Role: postgresql
========================

This role combines a bunch of variables used by other Postgres related roles.

Supported OS:
-------------
* RedHat
* CentOS
* OracleLinux

Requirements
------------

None

Role Variables
--------------

Available variables are listed below, along with default values (see `defaults/main.yml`):

    # Clusters and Databases to ensure exist.
    postgresql_clusters: []

    # postgres OS user and group
    postgresql_user: postgres
    postgresql_group: postgres

    # postgres service name
    postgresql_service_name: 'dbpostgres'

    # psql environment options used while connecting to a database
    postgresql_env_pgoptions: 'env PGOPTIONS="-c log_statement=none -c log_min_messages=panic -c log_min_error_statement=panic"'

    # postgres backup user
    postgresql_backup_user: backup

    # postgres nagios user
    postgresql_nagios_user: nagios

    # datavg and binvg mountpoint prefixes
    postgresql_datavg_mountpoint_prefix: /postgres/data
    postgresql_binvg_mountpoint_prefix: /postgres/apps
    postgresql_backupvg_mountpoint_prefix: /postgres/backup

    # parent directory for PostgreSQL homes
    postgresql_pghomes_directory: '{{ postgresql_binvg_mountpoint_prefix }}/homes'

    # prefix used in PostgreSQL home directories
    postgresql_pghomes_binary_prefix: 'binary-'

    # scripts directory
    postgresql_scripts_directory: /home/{{ postgresql_user }}/bin

    # password file name
    postgresql_passfile: '.pgpass'

    #
    # Software distribution
    #

    postgresql_binary_packages_directory: /software/postgres/binary

    postgresql_scripts_sql_directory: /home/postgres/sql
    postgresql_scripts_sql_log_directory: /home/postgres/sql/log

    #
    # HA config
    #
    postgresql_switchover_trigger_file: 'switchover.trigger.file'


Dependencies
------------

None

Example Playbook
----------------

`N/A`

This role is not used explicitly. It is referred in `meta.yml` in other PostgreSQL related roles.

Inside `vars/main.yml` or `group_vars/..` or `host_vars/..`:

    #---------------------------------------
    # overrides role 'postgresql' variables
    #---------------------------------------

    # parent directory for PostgreSQL homes
    postgresql_pghomes_directory: /postgres/apps/homes

    # prefix used in PostgreSQL home directories
    postgresql_pghomes_binary_prefix: 'foo-'

    # PostgreSQL clusters and databases
    postgresql_clusters:
      - name: CLUSTERNAME1 
        state: present     # (present|absent)
        version: 9.6.10
        port: 5432
        pgdata: # default '{{postgresql_datavg_mountpoint_prefix}}/{{name}}/instance'
        pghome: # default '{{postgresql_binvg_mountpoint_prefix }}/homes/foo-postgresql-{{ version }}'
        lc_collate:        # defaults to 'en_US.UTF-8'
        lc_ctype:          # defaults to 'en_US.UTF-8'
        encoding:          # defaults to 'UTF-8'
        use_global_hba_entries: true   # use entries defined in postgresql_instance_global_hba_entries variable
        custom_pg_hba_entries:
          - { type: host, database: replication, user: replicator, address: '192.168.132.91/32', auth_method: trust, comment: "Allow replication connections" }
          - { type: host, database: postgres, user: postgres, address: '192.168.132.91/32', auth_method: trust, comment: "pg_rewind switchover connections" }
        use_global_postgresql_auto_conf: true   # use settings from postgresql_instance_global_postgresql_auto_conf variable
        custom_postgresql_auto_conf: {}
          shared_preload_libraries: "'pg_stat_statements, pgsentinel'"
        ha_config:
          role: primary    # (primary|standby)
          primary_config:
            replication_slots:
              - 'standby_server.foo.com'
          standby_config:
            primary_ansible_inventory_hostname: 'standby_server.foo.com'
            pg_rewind_source_server: 'host=192.168.132.91 port=5432 user=postgres dbname=postgres'
            recovery_conf:
              primary_conninfo: "'user=replicator password=secret host=192.168.132.91 port=5432 sslmode=prefer sslcompression=1'"
              primary_slot_name: "'primary_server.foo.com'"
        filesystem:
          - { mntp_suffix: 'wals', lvsize: 5G, fstype: ext4, mode: '0755' }
          - { mntp_suffix: '', lvsize: 1G, fstype: ext4, mode: '0755' }
        postgres_exporter_config:   # Prometheus config
          state: present            # defaults to 'present'  (present|absent)
        users:
          - name: replicator
            password: md5XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
            role_attr_flags: 'NOSUPERUSER INHERIT NOCREATEROLE NOCREATEDB LOGIN REPLICATION NOBYPASSRLS'
            priv: # defaults to not set
            state: present # 'present|absent'
          - name: "{{ postgresql_backup_user|default('backup',true) }}"
            password: md5XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
            role_attr_flags: 'NOSUPERUSER INHERIT NOCREATEROLE NOCREATEDB LOGIN REPLICATION NOBYPASSRLS'
            state: present # 'present|absent'
          - name: nagios
            password: md5XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
            password_unencrypted: 'secret'
            role_attr_flags: 'NOSUPERUSER INHERIT NOCREATEROLE NOCREATEDB LOGIN NOREPLICATION NOBYPASSRLS'
            state: present # 'present|absent'
        databases:
          - name: NEXT_TAO        # required; the rest are optional
            owner: NEXT_TAO_dbo   # defaults to postgresql_user
            lc_collate:           # defaults to 'en_US.UTF-8'
            lc_ctype:             # defaults to 'en_US.UTF-8'
            encoding:             # defaults to 'UTF-8'
            tablespaces:
              - { name_suffix: DATA, lvsize: 50G, fstype: ext4, mode: '0755', default_tablespace: true }
              - { name_suffix: TEMP, lvsize: 5G, fstype: ext4, mode: '0755', temp_tablespaces: true }
            pgwatch2_config:
              preset_metrics: standard
              is_enabled: false
            state: present     # defaults to 'present'  (present|absent)

    # ... etc ...


License
-------

GPLv3 - GNU General Public License v3.0

Author Information
------------------

This role was created in 2018 by [Krzysztof Lewandowski](mailto:Krzysztof.Lewandowski@fastmail.fm).


