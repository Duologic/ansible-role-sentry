Duologic.sentry
===============

[![Build Status](https://travis-ci.org/Duologic/ansible-role-sentry.svg?branch=master)](https://travis-ci.org/Duologic/ansible-role-sentry)

This role configures and installs Sentry with Python.

Requirements
-----------

You probaly want Postgresql and Redis to run Sentry, see Example Playbook

NOTE: Provide the machine with enough memory (2GB+) as Sentry upgrade has a memory leak.
(see [getsentry/sentry#8862](https://github.com/getsentry/sentry/issues/8862))

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

You might want to set whether to run [cleanup](https://docs.sentry.io/server/cli/cleanup/)
and the number of days to keep data for by setting:
```yml
sentry_schedule_cleanup: true # whether to schedule a cleanup. Default is true
sentry_cleanup_days: 30 # the number of days to keep old data
```
Note that this does not delete metadata such as orgnanization and project configurations.


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

Supported distributions
-----------------------

This project is tested on CentOS 7, Debian 9 and Ubuntu 18.04.

Known issues with other distributions:

- CentOS 6: python2.7 not available
- Ubuntu 16.04: redis-server not available
- Debian 8: issues with cryptography (and possibly setuptools)

See this [build](https://travis-ci.org/Duologic/ansible-role-sentry/builds/487380995).

License
-------

MIT

Author Information
------------------

Jeroen Op 't Eynde, jeroen@simplistic.be
