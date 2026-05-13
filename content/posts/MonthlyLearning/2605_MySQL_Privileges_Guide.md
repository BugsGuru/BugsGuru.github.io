---
title: "MySQL 权限管理实战：创建用户、改密、授权与权限排查"
date: 2026-05-13
categories: [learning note]
tags: ["MySQL", "Database", "Privilege"]
keywords: ["CREATE USER", "ALTER USER", "GRANT", "SHOW GRANTS", "系统权限", "对象权限"]
description: "一篇覆盖 MySQL 用户创建、密码修改、权限授予与权限查询的实战指南，并说明 5.7 与 8.0 的关键差异"
---

## 背景

日常运维 MySQL 时，权限问题通常集中在四类操作：

- 创建用户
- 修改密码
- 赋予权限（系统级、库级、表级等）
- 排查“这个用户到底拥有哪些权限”

这篇文章以 MySQL 8.0 为主，补充 5.7 差异，给出可直接执行的命令模板。

---

## 1) 创建用户

## 1.1 基本语法

```sql
CREATE USER 'app_user'@'10.%' IDENTIFIED BY 'StrongPassword!2026';
```

说明：

- `'app_user'@'10.%'` 表示用户名 + 来源主机
- `10.%` 表示允许从 `10.x.x.x` 网段登录
- 如果只允许本机，可使用 `'app_user'@'localhost'`

## 1.2 推荐做法

1. 避免使用 `'%'` 全开放来源，优先收敛主机范围。  
2. 密码遵守复杂度策略（长度、大小写、数字、特殊字符）。  
3. 账号按职责拆分，例如读写账号、只读账号、运维账号，不要共用。

---

## 2) 修改密码

## 2.1 修改当前登录用户密码

```sql
ALTER USER USER() IDENTIFIED BY 'NewStrongPassword!2026';
```

## 2.2 管理员修改指定用户密码

```sql
ALTER USER 'app_user'@'10.%' IDENTIFIED BY 'AnotherStrongPassword!2026';
```

## 2.3 MySQL 版本差异

- **MySQL 8.0**：推荐统一使用 `ALTER USER ... IDENTIFIED BY ...`。  
- **MySQL 5.7**：同样支持 `ALTER USER`（5.7.6+）；老环境中可能还存在 `SET PASSWORD` 的历史写法，建议迁移到 `ALTER USER` 以统一维护。  

---

## 3) 赋予权限（含 MySQL 权限介绍）

MySQL 权限可以理解为“作用范围 + 权限类型”两个维度。

## 3.1 作用范围（Scope）

1. **全局级（Global）**：`*.*`，作用于整个实例。  
2. **库级（Database）**：`db_name.*`，作用于某个数据库。  
3. **表级（Table）**：`db_name.table_name`，作用于某张表。  
4. **列级（Column）**：仅某些列可 `SELECT/UPDATE`。  
5. **存储过程级（Routine）**：针对 `PROCEDURE/FUNCTION` 的 `EXECUTE` 等权限。  

## 3.2 常见权限类型

- **DML**：`SELECT`, `INSERT`, `UPDATE`, `DELETE`
- **DDL**：`CREATE`, `ALTER`, `DROP`, `INDEX`
- **管理类**：`PROCESS`, `RELOAD`, `SHOW DATABASES`, `REPLICATION SLAVE`, `REPLICATION CLIENT`
- **授权类**：`GRANT OPTION`（允许用户把自己已有权限再授予他人）

> 实际生产中，优先最小权限原则：只给“够用”的权限，不给“可能以后用到”的权限。

## 3.3 授权示例

### A. 给某库只读权限

```sql
GRANT SELECT ON reporting.* TO 'report_user'@'10.%';
```

### B. 给某库读写权限

```sql
GRANT SELECT, INSERT, UPDATE, DELETE ON app_db.* TO 'app_user'@'10.%';
```

### C. 给某张表的精细权限

```sql
GRANT SELECT, UPDATE ON app_db.orders TO 'ops_user'@'10.%';
```

### D. 允许转授权（谨慎）

```sql
GRANT SELECT ON app_db.* TO 'lead_user'@'10.%' WITH GRANT OPTION;
```

## 3.4 刷新权限是否还需要 `FLUSH PRIVILEGES`

- 使用 `CREATE USER`/`ALTER USER`/`GRANT`/`REVOKE` 这类账户管理 SQL 时，MySQL 会自动加载权限，**通常不需要**再执行 `FLUSH PRIVILEGES`。  
- 只有手工改系统表（不推荐）等特殊场景才需要考虑刷新。  

---

## 4) 查看用户拥有的所有权限

## 4.1 查看授权语句（最常用）

```sql
SHOW GRANTS FOR 'app_user'@'10.%';
```

这条命令会直接返回该用户的授权结果（包括全局、库级、表级等），是排查权限问题的首选。

## 4.2 查看当前登录用户权限

```sql
SHOW GRANTS;
```

等价于查看当前会话账号的授权。

## 4.3 从系统视图按维度排查（系统权限/表权限等）

在 MySQL 8.0 中，建议优先使用 `information_schema` 视图做细粒度排查：

```sql
-- 系统（全局）权限
SELECT * FROM information_schema.USER_PRIVILEGES
WHERE GRANTEE = "'app_user'@'10.%'";
```

```sql
-- 库级权限
SELECT * FROM information_schema.SCHEMA_PRIVILEGES
WHERE GRANTEE = "'app_user'@'10.%'";
```

```sql
-- 表级权限
SELECT * FROM information_schema.TABLE_PRIVILEGES
WHERE GRANTEE = "'app_user'@'10.%'";
```

```sql
-- 列级权限
SELECT * FROM information_schema.COLUMN_PRIVILEGES
WHERE GRANTEE = "'app_user'@'10.%'";
```

说明：

- `GRANTEE` 的格式是带引号的字符串，例如 `"'user'@'host'"`。  
- `SHOW GRANTS` 适合快速查看；`information_schema` 适合程序化审计与报表。  

---

## 5) MySQL 5.7 与 8.0 常见差异总结

1. **账号/密码管理语法**  
   - 8.0 更强调 `CREATE USER`/`ALTER USER` 标准化管理；  
   - 5.7 老版本环境可能仍残留历史语法（如 `SET PASSWORD`），建议统一迁移。  

2. **权限元数据与系统表实现差异**  
   - 8.0 数据字典与系统表实现有较大演进；  
   - 跨版本脚本尽量用 `SHOW GRANTS` 与 `information_schema`，避免直接依赖底层系统表结构。  

3. **认证插件与默认行为可能不同**  
   - 不同小版本可能默认认证插件不一致（例如与客户端兼容性相关）；  
   - 升级或混用客户端前，先验证认证方式与连接参数。  

---

## 6) 一套可复用的最小权限流程

```sql
-- 1) 创建用户
CREATE USER 'app_user'@'10.%' IDENTIFIED BY 'StrongPassword!2026';

-- 2) 授予最小必要权限
GRANT SELECT, INSERT, UPDATE, DELETE ON app_db.* TO 'app_user'@'10.%';

-- 3) 核验权限
SHOW GRANTS FOR 'app_user'@'10.%';
```

上线前再做两步：

- 用该账号真实连接应用，执行关键 SQL 验证权限是否“够用但不超配”  
- 把授权命令纳入变更记录，便于审计与回滚

---

## 参考

- [MySQL 8.0 Reference Manual - Account Management Statements](https://dev.mysql.com/doc/refman/8.0/en/account-management-statements.html)
- [MySQL 8.0 Reference Manual - GRANT Statement](https://dev.mysql.com/doc/refman/8.0/en/grant.html)
- [MySQL 8.0 Reference Manual - SHOW GRANTS Statement](https://dev.mysql.com/doc/refman/8.0/en/show-grants.html)
