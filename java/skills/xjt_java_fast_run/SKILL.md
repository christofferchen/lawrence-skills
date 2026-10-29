---
name: xjt_java_fast_run
description: 通用 Spring Boot 多模块 Maven 项目免打包快速启动。修改代码后无需 mvn package，直接基于 target/classes 与 Maven 依赖树启动；自动探测启动模块与 @SpringBootApplication 主类，依赖解析基于源码 pom，无需 mvn install。Use when asked to run/start a Spring Boot app locally without repackaging, refresh the classpath cache, or debug startup classpath issues.
---

# Spring Boot 多模块免打包快速启动（全局）

## 脚本位置

| 平台 | 命令 |
| --- | --- |
| Windows PowerShell | `C:\Users\lawrence\myconfigs\tools\spring-run.ps1` |
| Windows CMD | `C:\Users\lawrence\myconfigs\tools\spring-run.cmd` |
| Linux / macOS / Git Bash | `C:\Users\lawrence\myconfigs\tools\spring-run` |

`myconfigs/tools` 已加入 PATH 时，可直接调用 `spring-run`。

## 核心机制

1. **增量编译**：按需执行 `mvn compile`，不执行 `mvn package`。
2. **依赖解析**：`mvn dependency:build-classpath -pl :<module> -am -DincludeScope=runtime`
   - `-am` 让 Maven 把内部依赖模块纳入 reactor，**按源码 pom 解析依赖树**，因此新增依赖后**不需要** `mvn install`。
   - 缓存文件位于 `<module>/.run/classpath.txt`。
3. **类路径替换**：把本项目各模块的 jar 替换为对应的 `target/classes`，改代码后 `mvn compile` 即可生效。
4. **启动**：通过 Java 9+ `@argfile` 传递超长 classpath，规避 Windows 命令行长度限制。

## 常用命令

```powershell
# 最简：项目只有一个启动类时自动探测
spring-run.ps1

# 指定 profile / 端口
spring-run.ps1 -Profile dev -Port 8080

# 多启动类项目必须指定模块
spring-run.ps1 -Module my-boot

# 启动前增量编译
spring-run.ps1 -Module my-boot -Compile

# 编译指定模块
spring-run.ps1 -Module my-boot -Compile other-module

# pom.xml 变更后刷新依赖缓存
spring-run.ps1 -Module my-boot -UpdateDeps

# 远程调试 / JRebel
spring-run.ps1 -Module my-boot -RemoteDebug -DebugPort 5005
spring-run.ps1 -Module my-boot -Jrebel

# 追加自定义 JVM 参数
spring-run.ps1 -Module my-boot -ExtraJvmArgs "-Dfoo=bar","-Dbaz=1"
```

```bash
# Linux / macOS
spring-run --module my-boot --profile dev --port 8080
spring-run --module my-boot --update-deps
spring-run --module my-boot --compile
```

## 参数速查

| PowerShell | Bash | 说明 |
| --- | --- | --- |
| `-ProjectRoot <dir>` | `--project-root <dir>` | 项目根目录，默认当前目录 |
| `-Module <name>` | `--module <name>` | 启动模块目录名或 artifactId |
| `-MainClass <class>` | `--main-class <class>` | 主类全限定名，默认自动探测 |
| `-Profile <name>` | `--profile <name>` | Spring profile |
| `-Port <port>` | `--port <port>` | 服务端口 |
| `-MavenProfiles <p>` | `--maven-profiles <p>` | Maven `-P` 参数 |
| `-JavaHome <dir>` | `--java-home <dir>` | JDK 根目录，默认 `JAVA_HOME` |
| `-UpdateDeps` | `--update-deps` / `-u` | 强制刷新 classpath 缓存 |
| `-Compile` | `--compile` / `-c` | 启动前增量编译启动模块 |
| `-CompileModule <m>` | `--compile-module <m>` | 配合 `-Compile` 编译指定模块 |
| `-CompileAll` | `--compile-all` | 编译全部模块 |
| `-RemoteDebug` | `--debug` / `-d` | 开启 JDWP 调试 |
| `-DebugPort <port>` | `--debug-port <port>` | 调试端口，默认 5005 |
| `-Jrebel` | `--jrebel` | 启用 JRebel |
| `-ExtraJvmArgs <args>` | `--jvm-arg <arg>` | 追加 JVM 参数，可重复 |
| `-AppArgs <args>` | `--arg <arg>` | 追加应用参数，可重复 |

## traffic 项目示例

traffic 是典型的多启动类工程（18 个 `@SpringBootApplication`），必须用 `-Module` 指定启动模块：

```powershell
$env:JAVA_HOME="C:\Java\jdk1.8.0_202"

spring-run.ps1 `
  -ProjectRoot C:\Users\lawrence\work\gitlib\roadseg\traffic `
  -Module traffic-boot `
  -Profile local-kj `
  -Port 9999 `
  -MavenProfiles "boot,traffic-alarm,traffic-cockpit,traffic-emerge,traffic-guardian,traffic-wind,traffic-pub,traffic-knowledge,traffic-melon,traffic-tunnel" `
  -Compile
```

> 若 `traffic-boot` 模块已不存在，请替换为实际启动模块；脚本报错时会列出全部候选模块。

## 何时需要 `-UpdateDeps`

满足以下任一条件时执行：

- 修改了 `pom.xml`（新增/删除依赖、升级版本）
- 切换 Maven profile 导致依赖发生变化
- 启动时报 `NoClassDefFoundError` / `ClassNotFoundException`

日常只改业务代码时**不需要**，直接 `-Compile` 或直接运行即可。

## 故障排查

- **`无法加载文件 ... 未进行数字签名`（Windows PowerShell 5.1）**
  通常是项目目录被标记为 Low Integrity，而非真正的执行策略问题。执行：
  ```powershell
  icacls <项目目录> /setintegritylevel Medium /T /C /Q
  ```
- **`Unknown lifecycle phase ".test.skip=true"`**
  旧脚本未给 `-D...` 参数加引号导致；本脚本已修复。
- **`NoClassDefFoundError`**
  先执行 `-UpdateDeps` 刷新依赖缓存。
- **端口占用**
  清理旧 java 进程，或用 `-Port` 换端口。
- **PowerShell 5.1 解析中文报错**
  `.ps1` 文件必须保存为 UTF-8 with BOM；本脚本已带 BOM。

## 已废弃

项目内的 `run-local.ps1` / `run-local.sh` 已删除，统一使用本全局脚本。
