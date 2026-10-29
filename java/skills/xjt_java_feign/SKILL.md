---
name: xjt_java_feign
description: 按照 xjt 平台规范，在 Java/Spring Cloud 项目中跨模块调用时生成符合规范的 FeignClient 接口。不适用于前端、Go 等非 Java 项目。
---
# FeignClient 接口生成指南

## 1. 存放位置
必须放在被调用模块的 `*-api` 子模块 `feign` / `client` 包下。

## 2. 注解规范
- `@FeignClient`：必须指定 `contextId`、`value`（服务名）、`path`（含 `/inner` 前缀）
- 接口/方法上加 `@NoToken`（跳过Token校验）和 `@Inner`（内部调用标记）

## 3. 实现示例
```java
@FeignClient(contextId = "remoteExampleService", value = "example-biz", path = "/inner/example")
public interface RemoteExampleService {

    @NoToken @Inner
    @GetMapping("/{id}")
    R<ExampleVo> getById(@PathVariable("id") Long id);

    @NoToken @Inner
    @PostMapping("/list-by-ids")
    R<List<ExampleVo>> listByIds(@RequestBody List<Long> ids);
}
```

## 4. 调用端规范
- 注入：`private final` + `@RequiredArgsConstructor`
- 处理结果：`ResultUtils.getDataOrThrow(r)`
