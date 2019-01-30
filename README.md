Duologic.sentry
===============

This role configures and installs Sentry with Python.

Requirements
-----------

You probaly want Postgresql and Redis to run Sentry, see Example Playbook

Role Variables
--------------

You probably want to generate a strong secret key. See [defaults/main.yml](defaults/main.yml) for more configuration options.

 sentry_secret_key: 'UNSAFE'

Example Playbook
----------------

```
---
- become: true
  hosts: servers
  tasks:
    - name: Enable EPEL repo on EL for Redis role
      yum: pkg=epel-release state=present
      when: ansible_os_family == 'RedHat'
    - import_role:
        name: geerlingguy.redis
    - import_role:
        name: Duologic.postgresql_repository
      vars:
        postgres_repo_version: '9.5'
    - import_role:
        name: geerlingguy.postgresql
      vars:
        postgresql_hba_entries:
            - {type: local, database: sentry, user: sentry, auth_method: trust}
            - {type: local, database: all, user: postgres, auth_method: peer}
        postgresql_databases:
            - name: sentry
        postgresql_users:
            - name: sentry
              db: sentry
              role_attr_flags: SUPERUSER
    - name: Flush handlers so services are restarted before Sentry installation
      meta: flush_handlers
    - import_role:
        name: sentry
      vars:
          sentry_db_user: 'sentry'
          sentry_secret_key: 'SAFE'
```

License
-------

MIT

Author Information
------------------

Jeroen Op 't Eynde, jeroen@simplistic.be
