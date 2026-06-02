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
- `community.postgresql` collection and `psycopg2` on the host — only when
  `pgbouncer_manage_auth_query_function: true` (see Auth below).

## Role variables

See [`defaults/main.yml`](defaults/main.yml) for the full list. Most important groups:

| Group | Examples | Notes |
|---|---|---|
| Network | `pgbouncer_listen_addr`, `pgbouncer_listen_port` | Defaults: `0.0.0.0:6432`. |
| Auth | `pgbouncer_auth_type`, `pgbouncer_auth_user`, `pgbouncer_auth_query`, `pgbouncer_userlist`, `pgbouncer_manage_auth_query_function` | `userlist` is a list of `{user, password}` dicts. |
| Pooling | `pgbouncer_pool_mode`, `pgbouncer_max_client_conn`, `pgbouncer_default_pool_size`, … | Defaults to transaction pooling, 10000 max client conns. |
| Databases | `pgbouncer_databases` | List of `{name, dbname, host, port, pool_size, pool_mode}`. Use `name: "*"` for the catch-all entry. |
| Extras | `pgbouncer_extra_settings` | Free-form dict merged last into the `[pgbouncer]` section — overrides everything else. |

### auth_query function (`user_search`)

The default `pgbouncer_auth_query` looks up passwords via a `user_search()`
function, which PgBouncer needs to authenticate any role that is **not** listed
in `userlist.txt`. The function must exist in `pgbouncer_auth_dbname`, be
`SECURITY DEFINER` (to read `pg_shadow`), and grant `EXECUTE` only to
`pgbouncer_auth_user`.

Set `pgbouncer_manage_auth_query_function: true` to have the role create and
grant it for you. This requires a local PostgreSQL reachable by the `postgres`
OS user via peer auth and `psycopg2` on the host — typical when PgBouncer runs
on the same VM as the database. For split-host setups leave it `false` (the
default) and create `user_search()` next to the database yourself.

## Example usage

`requirements.yml`:

```yaml
roles:
  - name: okassov.pgbouncer
    version: v0.1.4
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
