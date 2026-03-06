# 快速开始指南

## 项目结构

```
your-db/
├── schema/
│   ├── db.changelog-master.xml   ← 只做 include，不动
│   └── db.changelog.xml          ← 所有 changeset 写这里
├── script/
│   ├── db-create                 ← 初始化数据库
│   ├── db-drop                   ← 删库
│   ├── db-reset                  ← drop + create
│   ├── db-update                 ← 增量更新
│   └── liquibase-wrapper         ← 读 .env 调用 liquibase CLI
├── .env.example                  ← 配置模板（提交到 git）
├── .gitignore                    ← .env 已加入忽略
├── liquibase.properties          ← Liquibase 配置文件
├── liquibase.pro                 ← Liquibase Pro 配置（可选）
└── QUICKSTART.md                 ← 本文件
```

## 1️⃣ 初始化配置

### 步骤 1：复制配置文件

```bash
# 从模板创建 .env 文件
cp .env.example .env
```

### 步骤 2：编辑 `.env` 文件

填入你的实际数据库连接信息：

```bash
LIQUIBASE_DATABASE_URL=jdbc:mysql://localhost:3306/your_database?useSSL=false&serverTimezone=UTC&allowPublicKeyRetrieval=true
LIQUIBASE_USERNAME=root
LIQUIBASE_PASSWORD=your_password
```

### 步骤 3：使脚本可执行（Linux/macOS）

```bash
chmod +x script/db-create
chmod +x script/db-drop
chmod +x script/db-reset
chmod +x script/db-update
chmod +x script/liquibase-wrapper
```

## 2️⃣ 常见操作

### 初始化数据库

```bash
./script/db-create
```

这会执行 `db.changelog.xml` 中的所有 changeset，初始化数据库。

### 增量更新

```bash
./script/db-update
```

执行新增的 changeset（之前执行过的不会重复执行）。

### 重置数据库

```bash
./script/db-reset
```

删除所有表后重新初始化，等于 `db-drop` + `db-create`。

### 删除数据库

```bash
./script/db-drop
```

删除数据库中的所有表和 Liquibase 元数据表。

## 3️⃣ 编写 Changeset

所有数据库变更都写在 `schema/db.changelog.xml` 文件中。

### 创建表示例

```xml
<changeSet id="2026-03-06-120000-create-users-table" author="admin">
    <createTable tableName="users">
        <column name="id" type="BIGINT" autoIncrement="true">
            <constraints primaryKey="true" nullable="false"/>
        </column>
        <column name="username" type="VARCHAR(255)">
            <constraints nullable="false" unique="true"/>
        </column>
        <column name="email" type="VARCHAR(255)">
            <constraints nullable="false" unique="true"/>
        </column>
        <column name="created_at" type="TIMESTAMP" defaultValueDb="CURRENT_TIMESTAMP"/>
        <column name="updated_at" type="TIMESTAMP" defaultValueDb="CURRENT_TIMESTAMP"/>
    </createTable>
</changeSet>
```

### 添加列示例

```xml
<changeSet id="2026-03-06-140000-add-phone-column" author="admin">
    <addColumn tableName="users">
        <column name="phone" type="VARCHAR(20)"/>
    </addColumn>
</changeSet>
```

### 创建索引示例

```xml
<changeSet id="2026-03-06-150000-add-user-email-index" author="admin">
    <createIndex indexName="idx_user_email" tableName="users">
        <column name="email"/>
    </createIndex>
</changeSet>
```

## 4️⃣ Changeset 命名规范

### ID 格式

```
YYYY-MM-DD-HHmmss-描述
```

**示例：**
- `2026-03-06-120000-create-users-table` - 创建用户表
- `2026-03-06-140000-add-email-column` - 添加邮箱列
- `2026-03-07-100000-create-orders-table` - 创建订单表

### Author 字段

使用开发者名字或团队名称，例如：
- `admin`
- `dev-team`
- `alice`

## 5️⃣ Windows 用户说明

Windows 上无法直接运行 bash 脚本，有以下解决方案：

### 方案 A：使用 WSL 或 Git Bash

如果安装了 WSL 或 Git Bash，可以直接运行脚本：

```bash
./script/db-create
```

### 方案 B：直接调用 liquibase 命令

```bash
# 读取 .env 文件中的变量，然后调用 liquibase
liquibase --url=jdbc:mysql://localhost:3306/your_database --username=root --password=your_password --changeLogFile=schema/db.changelog-master.xml update
```

### 方案 C：创建 PowerShell 脚本版本

可以创建 `.ps1` 版本的脚本，类似 Linux shell 脚本的功能。

## 6️⃣ 常见问题

### Q: 如何查看待执行的 SQL？

```bash
./script/liquibase-wrapper updateSQL
```

### Q: 如何回滚最后一个 changeset？

```bash
./script/liquibase-wrapper rollback 1
```

### Q: 如何跳过某个 changeset？

在 changeset 中添加 `<rollback />` 标签，或在 changeset 中添加 `<preconditions onFail="MARK_RAN">` 来跳过执行。

### Q: 数据库中的 DATABASECHANGELOG 表是什么？

这是 Liquibase 自动创建的元数据表，用来跟踪已执行的 changeset。不要手动修改它。

### Q: 如何在团队中협作？

1. 每个开发者创建自己的 changeset
2. 所有 changeset 追加到 `db.changelog.xml`
3. 提交到 git（但 `.env` 不提交，因为在 `.gitignore` 中）
4. 团队成员同步 `db.changelog.xml`，然后运行 `./script/db-update`

## 7️⃣ 相关资源

- [Liquibase 官方文档](https://docs.liquibase.com)
- [XML 格式参考](https://docs.liquibase.com/concepts/about-change-types.html)
- [Liquibase 命令行工具](https://docs.liquibase.com/tools-integrations/community-supported/liquibase-command-line.html)

---

**最后更新：** 2026-03-06
