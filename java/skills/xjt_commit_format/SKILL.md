---
name: xjt_commit_format
description: 按照个人提交规范，自动对本次 Git 提交信息进行格式化处理。适用于任意仓库。
---
# Git 提交规范

## 格式
```
<type>(<scope>): <subject>

<body>
```

- **type**: `feat` | `fix` | `docs` | `style` | `refactor` | `perf` | `test` | `chore`
- **scope**: 模块名（英文），如 `order-service`
- **subject**: 简短描述（中文）

## 核心规则
- 仅提交本次会话修改的内容
- 严禁 `git add .` 或 `git add --all`，必须显式 `git add <file1> <file2>...`
- SQL 迁移脚本若未被 .gitignore 排除，纳入提交；若被排除，**严禁** `git add -f`
- 可自动执行 `git commit`，**绝对禁止自动 `git push`**
- 严禁在提交信息中含个人标识或 AI 生成标记
- `git add` 与 `git commit` 合并为一次命令调用（用 `;` 连接），禁止拆分

## 示例
```text
feat(order-service): 增加模型字段默认值填充

1. 修改 DeviceServiceImpl 实现自动填充逻辑
2. 包含 db/migration 对应的数据库迁移脚本
```
