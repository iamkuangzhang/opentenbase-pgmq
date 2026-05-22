# OpenTenBase PGMQ

默认语言：中文  
English version: [English](#english)

OpenTenBase PGMQ 是基于 [PGMQ](https://github.com/pgmq/pgmq) 的 OpenTenBase 适配版插件。它以 SQL-only 插件载荷的形式提供，重点验证 PGMQ 在 OpenTenBase 分布式环境中的基础队列能力和插件生命周期治理。

这个仓库只放插件本身，不是 OpenTenBase PluginCtl 平台仓库。PluginCtl 可以治理、部署和验证它，但平台代码与插件代码保持分离。

## 当前状态

当前版本：`1.11.2-otb.0`

已在本地 OpenTenBase Docker 环境验证：

- 2 个 coordinator：`cn001`、`cn002`
- 2 个 datanode：`dn001`、`dn002`
- GTM 正常运行
- coordinator 连接入口：`127.0.0.1:30004`

已验证插件生命周期：

- install SQL 可成功执行
- `pgmq.create`
- `pgmq.send`
- `pgmq.read`
- `pgmq.archive`
- `pgmq.delete`
- `pgmq.metrics`
- `pgmq.drop_queue`
- 通过 `DROP SCHEMA pgmq CASCADE` 回滚
- removed probe 可验证移除状态

基础并发消费者验证已通过：

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

## 支持范围

当前 MVP 支持：

- SQL-only 安装
- 非分区 logged queue
- 通过 coordinator 执行队列操作
- 基础 OpenTenBase 分布式表验证
- PluginCtl manifest 与生命周期脚本

当前不支持，或尚不宣称完整支持：

- partitioned queue
- `pg_partman`
- topic routing 正确性
- unlogged queue
- Rust client 集成
- 完整分布式队列正确性证明
- 生产可用承诺

## 文件结构

```text
manifest.yml
payload/sql/install.sql
payload/sql/verify.sql
payload/sql/rollback.sql
LICENSE
THIRD_PARTY_NOTICES.md
docs/PGMQ_ADAPTATION_STUDY.md
```

## 手动安装

连接 OpenTenBase coordinator 后执行：

```bash
psql -h 127.0.0.1 -p 30004 -U opentenbase -d postgres -f payload/sql/install.sql
```

验证：

```bash
psql -h 127.0.0.1 -p 30004 -U opentenbase -d postgres -f payload/sql/verify.sql
```

在可控测试数据库中回滚：

```bash
psql -h 127.0.0.1 -p 30004 -U opentenbase -d postgres -f payload/sql/rollback.sql
```

## 配合 OpenTenBase PluginCtl 使用

本仓库包含可被 PluginCtl 识别的 `manifest.yml`：

```yaml
payload:
  source_root: payload
  install_sql: payload/sql/install.sql
  verify_sql: payload/sql/verify.sql
  rollback_sql: payload/sql/rollback.sql
```

PluginCtl 应该把这个仓库视为外部插件包，而不是平台内置代码。

## 许可证

本项目适配自 PGMQ。

- 上游仓库：https://github.com/pgmq/pgmq
- 上游许可证：PostgreSQL License
- 上游版权：Copyright (c) 2023, Tembo

上游许可证文本保存在 `LICENSE`。适配说明见 `THIRD_PARTY_NOTICES.md`。

---

## English

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
