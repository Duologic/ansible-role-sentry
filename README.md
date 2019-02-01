Duologic.sentry
===============

[![Build Status](https://travis-ci.org/Duologic/ansible-role-sentry.svg?branch=master)](https://travis-ci.org/Duologic/ansible-role-sentry)

This role configures and installs Sentry with Python.

Requirements
-----------

You probaly want Postgresql and Redis to run Sentry, see Example Playbook

Role Variables
--------------

You probably want to generate a strong secret key:

    sentry_secret_key: 'UNSAFE'

You can install extra packages with pip in the virtualenv, for example the plugin bundle:

    sentry_extra_pip_packages:
      - 'sentry-plugins==9.0.0'

The extra packages/plugins might need some extra configuration:

    sentry_extra_conf_py: |
        GITHUB_APP_ID = 'GitHub Application Client ID'
        GITHUB_API_SECRET = 'GitHub Application Client Secret'
        GITHUB_EXTENDED_PERMISSIONS = ['repo']

See [defaults/main.yml](defaults/main.yml) for more configuration options.

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
        sentry_extra_pip_packages:
            - 'sentry-plugins==9.0.0'
```

License
-------

MIT

Author Information
------------------

Jeroen Op 't Eynde, jeroen@simplistic.be
