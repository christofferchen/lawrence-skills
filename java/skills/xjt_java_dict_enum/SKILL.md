---
name: xjt_java_dict_enum
description: 在 Java/Spring 项目中同步添加通用枚举类，并生成数据字典变更 SQL。
---
# 字典枚举同步添加指南

新增选项型字段（如设备状态、故障类型）时，确保 Java 枚举与后台数据字典同步。

## 1. Java 枚举规范
注解：`@Getter`, `@AllArgsConstructor`；字段：`code`（`@EnumValue` + `@JsonValue`）和 `msg`；提供 `ofCode` / `ofMsg` 静态方法。

```java
@Getter
@AllArgsConstructor
public enum AssetTypeEnumSpec {
    DEVICE("DEVICE", "设备"),
    LOC("LOC", "设施");

    @EnumValue @JsonValue
    private final String code;
    private final String msg;

    public static AssetTypeEnumSpec ofCode(String code) {
        return Arrays.stream(values()).filter(i -> Objects.equals(i.getCode(), code)).findFirst().orElse(null);
    }
    public static AssetTypeEnumSpec ofMsg(String msg) {
        return Arrays.stream(values()).filter(i -> Objects.equals(i.getMsg(), msg)).findFirst().orElse(null);
    }
}
```

## 2. 数据字典 SQL 添加
生成最简单的 MySQL DML 供用户在目标环境执行。严禁通过 MCP、Python 脚本、数据库客户端或其他方式直接连接或写入数据库。

执行规则：

1. 枚举放在 api/api-core 模块相关包下。
2. 确保 Java `code` = 字典项 `item_value`，`msg` = `label`。
3. SQL 统一使用 `tenant_id = 1`。
4. 先分别查询当前环境的最大 ID，再把具体数值写入 DML，DML 内不使用 SQL 变量、`FROM DUAL` 或条件查询构造。
5. 先确认/新增 `sys_dict` 字典类型并记录其 ID，再维护 `sys_dict_item` 字典项。
6. 字典项 `sort_order` 按枚举声明顺序从 1 开始。
7. 已存在同 `dict_type + item_value` 的字典项时不要静默覆盖 `label`；如与枚举 `msg` 不一致，单独提示用户确认后再生成更新语句。
8. 优先使用可跨环境重复执行的幂等写法；不要执行 SQL，只生成变更语句。

## 3. 生成 SQL 前必须查询
在生成 DML 前，先向用户提供或确认以下查询在目标环境的执行结果：

```sql
SELECT IFNULL(MAX(id), 0) AS max_dict_id FROM sys_dict;
SELECT IFNULL(MAX(id), 0) AS max_item_id FROM sys_dict_item;
SELECT id, dict_type, description, del_flag
FROM sys_dict
WHERE dict_type = 'asset_type' AND tenant_id = 1;
```

说明：

1. 新增字典类型时，`sys_dict.id` 使用 `max_dict_id + 1`；字典类型已存在时，直接使用查询到的 ID。
2. 新增多个字典项时，按顺序使用 `max_item_id + 1`、`max_item_id + 2` 等；如果部分字典项已存在，只为缺失项分配新 ID。
3. ID 必须写成具体数值，确保同一份 DML 在各环境已执行过时重复执行不会新增重复数据。
4. 查询和 DML 由用户在目标环境执行，生成前应提醒用户避免与其他字典写入并发执行，防止 `MAX(id)` 取值冲突。

## 4. DML 模板
以下示例假设已查询到 `max_dict_id = 1000`、`max_item_id = 5000`，且字典类型和两个字典项均不存在。将占位的业务值替换为实际值后生成；ID 示例必须替换为目标环境查询结果计算出的具体数值。

```sql
INSERT IGNORE INTO sys_dict (id, dict_type, description, create_by, update_by, create_time, update_time, remarks, system_flag, del_flag, tenant_id)
VALUES (1001, 'asset_type', '资产类型', 'admin', 'admin', NOW(), NOW(), NULL, '0', '0', 1);

INSERT IGNORE INTO sys_dict_item (id, dict_id, item_value, label, dict_type, description, sort_order, remarks, tag_type, style, create_by, update_by, create_time, update_time, del_flag, tenant_id)
VALUES
    (5001, 1001, 'DEVICE', '设备', 'asset_type', '设备', 1, NULL, NULL, '', 'admin', 'admin', NOW(), NOW(), '0', 1),
    (5002, 1001, 'LOC', '设施', 'asset_type', '设施', 2, NULL, NULL, '', 'admin', 'admin', NOW(), NOW(), '0', 1);
```

## 5. 交付要点
1. 告知用户枚举已添加，并输出先查询最大 ID 的 SQL、后续 DML，以及每个 ID 的计算说明。
2. 如需沉淀为 Flyway 脚本，按项目命名规范保存到 `db/migration/`；脚本中同样使用提前查询得到的具体 ID 和简单 DML。
3. 不要执行 SQL；只生成变更语句。
