---
name: java-coding-standard
description: Use when reading, writing, or reviewing Java code (.java files, Maven or Gradle projects, Spring / MyBatis-Plus backends, JDK 8 codebases). Applies the user's personal Java coding standard. Do not use for Go, frontend, Python, or scripting tasks.
---

# Java 编码规范

本规范适用于所有 Java 项目；Go、前端、脚本类任务不适用。

## 通用代码质量

- 使用 Objects.equals 进行安全判等，使用 `!= null` 而不是 Objects.nonNull 进行非空判断
- 在确保可读性的前提下优先使用工具类，顺序为 Apache Commons、Guava、Hutool
- 使用 Functions.identity()，而不是 `a -> a`
- 改完代码后记得导包：使用 import 导入，避免使用全限定类名
- 日志输出使用占位符 `{}` 而非字符串拼接，如 `log.error("失败,id={}", id, e)`
- 空集合处理：使用 Collections.emptyList()、Collections.emptyMap() 代表空集合
- 集合处理优先使用 Stream API 而非 for 循环，使用 Collectors.toMap/toList/groupingBy 等；需要遍历时优先 for 而不是 .forEach()
- 对象转换：需要忽略赋值的字段较少时，优先 BeanUtil.copyProperties() 或 copyToList()，而不是逐个字段赋值

## MyBatis-Plus 技术栈

仅在项目使用 MyBatis-Plus 时适用。

- 优先采用 `this.lambdaQuery()` 和 `this.lambdaUpdate()`，提高可读性、安全性及可维护性
- 分页参数一律使用 MyBatis-Plus 的 `Page` 对象接收

## 注释规范

- Javadoc `@author` 填写 zhengyafei，不需要 `@date`

## 方法顺序

- 调用方法排前面，被调用方法紧跟在后面，按深度优先规则排序
- 业务方法顺序：查询方法 → 创建方法 → 更新方法 → 删除方法
