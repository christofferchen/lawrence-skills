---
name: xjt_java_crud
description: 按照 xjt 平台规范自动生成标准的 Controller -> Service -> Mapper -> Entity/VO 全套 CRUD 代码。适用于 Java/Spring Boot 的 biz 模块，不适用于前端、Go 等非 Java 项目。
---
# 标准 CRUD 模块生成指南

## 1. 整体架构规范
- **包路径**: `biz` 模块的 `controller`, `service`, `mapper`, `entity` 包下
- **层次关系**: Controller -> Service -> Mapper -> Entity
- **返回值**: 统一使用 `R<T>` 封装

## 2. Controller 规范
- **注解顺序**: `@Slf4j`, `@RequiredArgsConstructor`, `@RestController`, `@RequestMapping`, `@Tag`
- **依赖注入**: `private final` + `@RequiredArgsConstructor`
- **标准方法**: `POST /page`（分页）、`GET /{id}`（详情）、`POST /`（创建）、`PUT /`（更新）、`DELETE /{id}`（删除）
- **可选**: `POST /list`（全量列表，按需添加）
- update 参数：`@RequestBody @Validated(EditReq.Modify.class) EditReq req`（id 通过 req 传入，无路径变量）

```java
@PutMapping("/")
@Operation(summary = "更新")
public R<Void> updateExample(@RequestBody @Validated(EditExampleReq.Modify.class) EditExampleReq req) {
    exampleService.updateExample(req);
    return R.ok();
}
```

## 3. Service 层规范
- **接口**: 继承 `IService<Entity>`；**实现类**: 继承 `ServiceImpl<Mapper, Entity>`
- **方法命名**: `pageExample`, `getExampleDetail`, `createExample`, `updateExample`, `deleteExample`（`list` 可选）
- **数据库操作**: 单表禁止直接调用 baseMapper/Mapper，必须通过 Service 自身方法；优先 `this.lambdaQuery()` / `this.lambdaUpdate()`
- **list 复用 page**: 使用 `new Page<>(1, Long.MAX_VALUE)` + `setSearchCount(false)` + `setMaxLimit(-1L)`
- **关联填充 (fillXxx)**: 禁止循环查库。流程：去重收集ID → 批量查 → 转Map → 回填

```java
private void fillDeptName(List<ExampleVo> list) {
    Set<Long> ids = list.stream().map(ExampleVo::getDeptId).filter(Objects::nonNull).collect(Collectors.toSet());
    if (CollUtil.isEmpty(ids)) return;
    Map<Long, String> map = deptService.listByIds(ids).stream().collect(Collectors.toMap(Dept::getId, Dept::getName));
    list.forEach(i -> i.setDeptName(map.get(i.getDeptId())));
}
```

- **参数校验**: 重要参数使用 `@NonNull`（如 `@NonNull EditReq req`），内部无需重复判空
- 更新操作加 `@Transactional(rollbackFor = Exception.class)`

## 4. Mapper 规范
- 继承 `BaseMapper<Entity>`，加 `@Mapper` 注解

## 5. Entity / VO / Req 规范
- **VO**: 回显和详情；**QueryReq**: 查询条件；**EditReq**: 新增+修改共用（内嵌 `Create`/`Modify` 校验组）
- **注解**: `@Data`, `@NoArgsConstructor`, `@AllArgsConstructor`, `@SuperBuilder`, `@Accessors(chain = true)`
- **ID 校验**: `Modify` 组对 `id` 加 `@NotNull`，`Create` 组不校验

```java
public class EditExampleReq {
    @NotNull(groups = Modify.class)
    private Long id;
    @NotBlank(message = "名称不能为空")
    private String name;

    public interface Create extends Default {}
    public interface Modify extends Default {}
}
```
