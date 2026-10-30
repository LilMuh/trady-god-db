# 快速开始

这个仓库管理 `trady_god_db` 的表结构。数据库是 PostgreSQL 16，表结构变更由 Liquibase 执行。
交易程序 `tradyGod-api` 读写这个数据库。

## 目录

```
trady-god-db/
├── schema/
│   ├── db.changelog-master.xml   主 changelog，只引用 db.changelog.xml
│   └── db.changelog.xml          全部 changeset
├── script/                       调用本机 liquibase 命令的脚本
├── docker-compose.yml            启动数据库并执行 changeset
├── .env.example                  配置模板
└── liquibase.properties          直接使用 liquibase 命令时的默认配置
```

## 默认连接信息

| 项目 | 值 |
|---|---|
| 主机 | localhost |
| 端口 | 5435 |
| 数据库 | trady_god_db |
| 用户 | tradyUser |
| 密码 | tradyPassword |

`tradyGod-api/.env` 中的 `DB_HOST`、`DB_PORT`、`DB_NAME`、`DB_USER`、`DB_PASSWORD` 必须和这张表一致。

本机 5432 端口已经有其他 PostgreSQL，所以容器默认映射到 5435。要换端口，在 `.env` 中修改 `DB_HOST_PORT`，同时修改 `LIQUIBASE_DATABASE_URL` 和 `tradyGod-api` 的 `DB_PORT`。

## 方法 1：docker compose（推荐）

1. 创建配置文件：

   ```bash
   cp .env.example .env
   ```

2. 启动数据库，执行全部 changeset：

   ```bash
   docker compose up -d
   ```

3. 检查 changeset 执行结果：

   ```bash
   docker compose logs liquibase
   ```

   日志中有 `Liquibase command 'update' was executed successfully`，表示执行成功。

4. 增加新的 changeset 以后，再执行一次 update：

   ```bash
   docker compose run --rm liquibase
   ```

## 方法 2：本机 liquibase 命令

先安装 liquibase 命令行工具，然后：

```bash
cp .env.example .env
chmod +x script/*
./script/db-update
```

| 脚本 | 作用 |
|---|---|
| `script/db-create` | 执行全部 changeset |
| `script/db-update` | 执行新增的 changeset |
| `script/db-drop` | 删除全部表。执行前要求确认 |
| `script/db-reset` | 先删除全部表，再执行全部 changeset |
| `script/liquibase-wrapper` | 读取 `.env`，把参数传给 liquibase |

警告：`db-drop` 和 `db-reset` 会删除全部数据。

## 增加 changeset

1. 在 `schema/db.changelog.xml` 末尾追加一个新的 `<changeSet>`。
2. id 的格式：`YYYY-MMDD-描述`，例如 `2026-1030-orders-swap-columns`。
3. 执行 update。

警告：不要修改已经执行过的 changeset。Liquibase 会检查每个 changeset 的校验和，修改后 update 会失败。

## 常见问题

**查看将要执行的 SQL，不实际执行：**

```bash
./script/liquibase-wrapper updateSQL
```

**回滚最后一个 changeset：**

```bash
./script/liquibase-wrapper rollback-count 1
```

**DATABASECHANGELOG 表是什么：** Liquibase 用这张表记录已经执行过的 changeset。不要手动修改这张表。
