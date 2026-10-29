---
name: xjt_java_stat_query
description: 针对 Java/MyBatis-Plus 统计类需求，生成基于虚拟字段与 LambdaQuery 的分组统计代码。
---
# MyBatis-Plus 分组统计指南

按时间/部门/状态进行汇总统计时使用此规范。

## 1. VO 虚拟字段
在 VO 类中用 `@TableField(exist = false)` 定义接收统计结果的字段：
```java
@TableField(exist = false) private Integer count;   // 统计数量
@TableField(exist = false) private String groupKey; // 分组值（如日期或状态）
```

## 2. 查询规范
- 禁止手写 XML（复杂场景除外）
- 必须显式声明 `select()` 包含分组字段及聚合字段，并显式声明 `groupBy()`

```java
public List<OrderVo> statOrderByStatus() {
    return this.lambdaQuery()
        .select(Order::getStatus, "count(*) as count")
        .groupBy(Order::getStatus)
        .list()
        .stream()
        .map(i -> BeanUtil.copyProperties(i, OrderVo.class))
        .collect(Collectors.toList());
}
```
