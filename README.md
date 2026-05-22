# OpenTenBase PGMQ

OpenTenBase PGMQ is an OpenTenBase adaptation of [PGMQ](https://github.com/pgmq/pgmq), packaged as a SQL-only distributed plugin payload.

This repository is focused on the plugin itself. It can be governed and deployed by OpenTenBase PluginCtl, but it is intentionally separate from the PluginCtl platform repository.

## Status

Current version: `1.11.2-otb.0`

Validated locally on an OpenTenBase Docker environment with:

- 2 coordinators: `cn001`, `cn002`
- 2 datanodes: `dn001`, `dn002`
- GTM running
- coordinator entrypoint: `127.0.0.1:30004`

Verified plugin lifecycle:

- install SQL runs successfully
- `pgmq.create`
- `pgmq.send`
- `pgmq.read`
- `pgmq.archive`
- `pgmq.delete`
- `pgmq.metrics`
- `pgmq.drop_queue`
- rollback through `DROP SCHEMA pgmq CASCADE`
- removed probe

Basic concurrent consumer probe passed:

```text
expected_count=100
consumer_a_count=52
consumer_b_count=48
total_read=100
unique_read=100
duplicates_count=0
missing_count=0
unexpected_count=0
remaining_rows=0
```

## Scope

Supported in this MVP:

- SQL-only install
- non-partitioned logged queues
- coordinator-side queue operations
- basic OpenTenBase distributed table validation
- PluginCtl manifest and lifecycle scripts

Not yet supported or not yet claimed:

- partitioned queues
- `pg_partman`
- topic routing correctness
- unlogged queues
- Rust client integration
- full distributed queue correctness proof
- production readiness

## Files

```text
manifest.yml
payload/sql/install.sql
payload/sql/verify.sql
payload/sql/rollback.sql
LICENSE
THIRD_PARTY_NOTICES.md
docs/PGMQ_ADAPTATION_STUDY.md
```

## Install Manually

Run against an OpenTenBase coordinator connection:

```bash
psql -h 127.0.0.1 -p 30004 -U opentenbase -d postgres -f payload/sql/install.sql
```

Verify:

```bash
psql -h 127.0.0.1 -p 30004 -U opentenbase -d postgres -f payload/sql/verify.sql
```

Rollback in a controlled test database:

```bash
psql -h 127.0.0.1 -p 30004 -U opentenbase -d postgres -f payload/sql/rollback.sql
```

## Use With OpenTenBase PluginCtl

This repository includes `manifest.yml` using portable paths:

```yaml
payload:
  source_root: payload
  install_sql: payload/sql/install.sql
  verify_sql: payload/sql/verify.sql
  rollback_sql: payload/sql/rollback.sql
```

PluginCtl integration should treat this as an external plugin package.

## License

This project is adapted from PGMQ.

- Upstream: https://github.com/pgmq/pgmq
- Upstream license: PostgreSQL License
- Upstream copyright: Copyright (c) 2023, Tembo

The upstream license text is included in `LICENSE`. See `THIRD_PARTY_NOTICES.md` for adaptation notes.
