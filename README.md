# ansible-role-pgbouncer

Minimal Ansible role for installing and configuring [PgBouncer](https://www.pgbouncer.org/) on Debian/Ubuntu hosts.

## What it does

- Installs the `pgbouncer` package from distro repositories.
- Creates and persists `/run/pgbouncer` (tmpfiles).
- Renders `/etc/pgbouncer/pgbouncer.ini` and `/etc/pgbouncer/userlist.txt` from your variables.
- Restarts (not reloads) PgBouncer on `pgbouncer.ini` changes.

## Requirements

- Ubuntu 22.04+ / Debian 12+ with the `pgbouncer` apt package available.
- Ansible 2.12+.

## Role variables

See [`defaults/main.yml`](defaults/main.yml) for the full list. Most important groups:

| Group | Examples | Notes |
|---|---|---|
| Network | `pgbouncer_listen_addr`, `pgbouncer_listen_port` | Defaults: `0.0.0.0:6432`. |
| Auth | `pgbouncer_auth_type`, `pgbouncer_auth_user`, `pgbouncer_auth_query`, `pgbouncer_userlist` | `userlist` is a list of `{user, password}` dicts. |
| Pooling | `pgbouncer_pool_mode`, `pgbouncer_max_client_conn`, `pgbouncer_default_pool_size`, … | Defaults to transaction pooling, 10000 max client conns. |
| Databases | `pgbouncer_databases` | List of `{name, dbname, host, port, pool_size, pool_mode}`. Use `name: "*"` for the catch-all entry. |
| Extras | `pgbouncer_extra_settings` | Free-form dict merged last into the `[pgbouncer]` section — overrides everything else. |

## Example usage

`requirements.yml`:

```yaml
roles:
  - name: okassov.pgbouncer
    version: v0.1.3
```

Playbook:

```yaml
- hosts: pgbouncer
  become: true
  roles:
    - role: okassov.pgbouncer
      vars:
        pgbouncer_databases:
          - name: cars
            dbname: cars
            host: 10.0.0.10
            port: 5432
            pool_size: 50
        pgbouncer_userlist:
          - user: pgbouncer
            password: "md5xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"
```

## License

MIT
