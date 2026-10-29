---
name: xjt_sql_migration
description: 生成符合项目规范的数据库迁移脚本（Flyway 格式）
---
# 数据库迁移脚本生成指南

## 1. 命名与存放
- 路径：`db/migration/`
- 命名：`V{YYYYMMDD}_{HHmm}_{description}.sql`（如 `V20240330_1530_add_device_column.sql`）
- `tenant_id` 统一设置为 `1`

## 2. 核心规范

### ID 计算（禁止硬编码）
```sql
SELECT @item_id_start := IFNULL(MAX(id), 0) FROM sys_dict_item;
INSERT INTO sys_dict_item (id, ...) VALUES (@item_id_start + 1, ...);
```

### 建表模板
```sql
CREATE TABLE table_name (
    id          BIGINT       NOT NULL COMMENT '主键ID' PRIMARY KEY,
    dept_id     BIGINT       NOT NULL DEFAULT 0 COMMENT '部门ID',
    create_user VARCHAR(50)  NOT NULL DEFAULT '' COMMENT '创建人',
    create_time DATETIME     NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
    update_user VARCHAR(50)  NOT NULL DEFAULT '' COMMENT '更新人',
    update_time DATETIME     NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',
    tenant_id   BIGINT       NOT NULL DEFAULT 1 COMMENT '所属租户',
    del_flag    CHAR(1)      NOT NULL DEFAULT '0' COMMENT '删除标志'
) COMMENT '表描述';
```

### 字段维护
- 新增：`ALTER TABLE t ADD COLUMN ...`
- 删除：注释掉删除语句保留痕迹（由人工线下处理）
- **禁止重命名列**，改为新增+删除

## 3. 字典模板
```sql
SELECT @dict_id := IFNULL(MAX(id), 0) + 1 FROM sys_dict;
INSERT INTO `sys_dict` (`id`, `dict_type`, `description`, `system_flag`, `tenant_id`, `create_time`, `del_flag`)
VALUES (@dict_id, 'your_type', '描述', '0', 1, NOW(), '0');

SELECT @item_id_start := IFNULL(MAX(id), 0) FROM sys_dict_item;
INSERT INTO `sys_dict_item` (`id`, `dict_id`, `item_value`, `label`, `dict_type`, `tenant_id`, `create_time`, `del_flag`)
VALUES
(@item_id_start + 1, @dict_id, 'VALUE1', '标签1', 'your_type', 1, NOW(), '0'),
(@item_id_start + 2, @dict_id, 'VALUE2', '标签2', 'your_type', 1, NOW(), '0');
```

> 执行前使用 `mysql_access` 查询相关表的 `MAX(id)` 确认起点。
