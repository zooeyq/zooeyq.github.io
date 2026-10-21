---
title: Maven 入门学习笔记：依赖管理、生命周期与 JUnit 测试
tags:
  - Java
  - Maven
  - JUnit
categories:
  - 技术笔记
date: 2026-10-19 23:47:00
---


今天学习 Maven 时，我记下了仓库、依赖、生命周期和测试等关键词。刚接触时，这些概念比较零散，所以我把它们整理成一篇笔记：先理解 Maven 为什么有用，再弄清依赖从哪里来、项目如何构建，最后用 JUnit 写一个小测试。

本文按 Maven 3 的常见用法讲解；代码示例固定使用 JDK 17 和 JUnit 5.11.1，便于练习，不代表最新版本。

<!-- more -->

## 一、Maven 是什么？

Maven 是一个项目构建和管理工具。对初学者来说，可以先记住它的三个作用。

### 1. 方便的依赖管理

以前使用第三方库，可能需要手动下载 JAR，再添加到项目中。如果这个库还依赖其他库，也要继续下载。

使用 Maven 后，只需要在 `pom.xml` 中声明依赖，Maven 就能根据配置解析、下载依赖，并将它们加入对应的编译或运行路径。

`pom.xml` 中的 POM 是 Project Object Model，即项目对象模型。它描述项目身份、依赖和构建配置等信息。

### 2. 统一的构建流程

Maven 把编译、测试、打包和发布构件等工作组织成统一流程。不同项目遵循相近的约定，切换项目时也更容易上手。

### 3. 约定的项目结构

```text
maven-demo/
├── pom.xml
├── src/
│   ├── main/
│   │   ├── java/          # 正式 Java 代码
│   │   └── resources/     # 正式配置和资源
│   └── test/
│       ├── java/          # 测试代码
│       └── resources/     # 测试资源
└── target/                # 构建输出
```

纠正一下我最初的记录：默认不是在项目根目录直接创建 `test` 和 `resources`，而是区分 `src/main` 和 `src/test`。

`target` 通常由构建过程生成，包括编译后的类、测试报告和打包结果。目录结构可以调整，但入门阶段遵循默认约定最方便。参见 [Maven 标准目录说明](https://maven.apache.org/guides/introduction/introduction-to-the-standard-directory-layout.html)。

## 二、JAR 和 WAR 是什么？

| 格式 | 含义 | 常见用途 |
| --- | --- | --- |
| JAR | Java Archive，Java 归档文件 | 分发 Java 类、资源和相关元数据 |
| WAR | Web Application Archive，Web 应用归档文件 | 将 Web 应用部署到 Servlet 容器 |

JAR 可以是供其他项目引用的类库，也可以配置成可执行程序，但不是所有 JAR 都能直接通过 `java -jar` 运行。

WAR 通常包含 Web 资源，以及 `WEB-INF/classes`、`WEB-INF/lib` 等目录。项目采用什么打包方式，可以通过 POM 中的 `<packaging>` 指定，默认是 `jar`。

## 三、Maven 的安装与配置

Windows 下的基本步骤是：

1. 安装 JDK，设置 `JAVA_HOME`。
2. 从官网下载 Maven 二进制压缩包并解压。
3. 将 Maven 的 `bin` 目录加入 `Path`。
4. 重新打开 CMD，检查安装结果。

```cmd
java -version
mvn -v
```

`mvn -v` 可以显示 Maven 版本、使用的 Java 版本等信息。安装步骤参见 [Maven 官方安装说明](https://maven.apache.org/install.html)。

### 本地仓库配置

默认的本地仓库是用户目录下的 `.m2/repository`。如果想换到其他磁盘，可以在 `%USERPROFILE%\.m2\settings.xml` 中设置：

```xml
<settings xmlns="http://maven.apache.org/SETTINGS/1.0.0"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0 https://maven.apache.org/xsd/settings-1.0.0.xsd">
  <localRepository>D:/maven-repository</localRepository>
</settings>
```

路径只是示例；如果文件已有其他配置，应合并到现有的 `<settings>` 中。

修改本地仓库位置、配置镜像都是可选步骤，不是安装成功的必要条件。

## 四、仓库是什么？依赖从哪里下载？

仓库保存构件及相关信息，例如 JAR、POM 和版本元数据。从 Maven 的角度看，主要分为本地仓库和远程仓库。

| 名称 | 作用 |
| --- | --- |
| 本地仓库 | 缓存下载的依赖，也保存通过 `mvn install` 安装的本地构件 |
| 中央仓库 | 默认使用的公共远程仓库，提供大量开源构件 |
| 企业私服 | 企业维护的远程仓库，保存内部构件，也可以代理公共仓库 |
| 公共镜像 | 按镜像规则替代某个远程仓库的访问地址 |

因此，我笔记里的“配置阿里云私服”更准确地说通常是“配置阿里云公共 Maven 镜像”。它和公司内部维护的私服不是同一个概念。

### 查找依赖的过程

对普通外部依赖，可以先这样理解：

```text
读取依赖声明
    ↓
检查本地可用的构件
    ↓ 必要时
访问有效配置中的远程仓库
    ↓
下载并缓存到本地仓库
```

没有额外配置时，远程仓库通常是 Central。配置了镜像，则按照 `mirrorOf` 的匹配规则访问镜像。

这里不是固定的“本地仓库 → 私服 → 中央仓库”。如果公司私服代理中央仓库，向上游取包的行为发生在私服端；镜像访问失败，也不保证自动回退到 Central。多模块构建还可能直接使用本次构建中的模块产物。

相关规则见 [仓库说明](https://maven.apache.org/guides/introduction/introduction-to-repositories.html) 和 [镜像配置说明](https://maven.apache.org/guides/mini/guide-mirror-settings.html)。

### 镜像配置需要注意什么？

镜像配置放在 `settings.xml` 的 `<mirrors>` 中。使用公共镜像时，应采用服务商提供的地址，并确认匹配范围：

- `mirrorOf` 为 `central`：只替换 Central。
- `mirrorOf` 为 `*`：匹配所有仓库，可能影响企业内部依赖的下载。

镜像不能让原本不存在的构件凭空出现。如果下载失败，还需要检查坐标、版本、仓库内容、网络以及更新策略。

## 五、Maven 坐标与版本

### 1. 坐标用来定位构件

常说的 GAV 坐标包含三个字段：

| 字段 | 含义 | 示例 |
| --- | --- | --- |
| `groupId` | 组织或项目组标识，常采用反向域名 | `org.junit.jupiter` |
| `artifactId` | 模块或构件名称 | `junit-jupiter` |
| `version` | 版本号 | `5.11.1` |

更完整的构件定位还会涉及类型和分类器，例如 `type`、`classifier`。

`scope` 表示依赖作用域，不属于 GAV 坐标。

### 2. SNAPSHOT 与正式版本

- `1.0-SNAPSHOT`：开发中的快照版本，同一个版本标识可能对应后续更新的内容，是否检查更新取决于仓库策略。
- `1.0.0`：非快照版本，通常作为正式发布版本，发布后应保持内容稳定。

“正式版本”不等于自动满足生产要求，仍然需要测试和项目自己的发布流程。

### 3. 引入依赖

在 POM 的 `<dependencies>` 中加入依赖声明，例如：

```xml
<dependency>
  <groupId>org.junit.jupiter</groupId>
  <artifactId>junit-jupiter</artifactId>
  <version>5.11.1</version>
  <scope>test</scope>
</dependency>
```

这表示引入指定版本的 JUnit Jupiter，并将它用于测试。

## 六、依赖作用域 scope

依赖不一定在所有阶段都需要。例如，测试框架通常不应该成为正式程序运行时的依赖。

下表针对当前项目直接声明的依赖：

| scope | 主代码编译 | 测试 | 主程序运行 | 常见用途 |
| --- | --- | --- | --- | --- |
| `compile`，默认值 | ✓ | ✓ | ✓ | 正式代码使用的普通类库 |
| `provided` | ✓ | ✓ | — | 运行环境提供的 Servlet API 等 |
| `runtime` | — | ✓ | ✓ | 通过标准接口加载的数据库驱动等 |
| `test` | — | ✓ | — | JUnit 等测试工具 |
| `system` | ✓ | ✓ | — | 通过 `systemPath` 指向本机文件，不推荐 |

`import` 是特殊作用域，只用于 `<dependencyManagement>` 中 `type` 为 `pom` 的依赖，以导入 BOM 的版本管理信息，不直接把库加入项目。

作用域也影响依赖传递。例如，当前项目的 `test`、`provided` 依赖通常不会继续作为传递依赖提供给下游项目。

此外，依赖可用不等于它会被放入最终归档：普通 JAR 默认不会把所有依赖打进同一个文件。详细规则参见 [Maven 依赖机制](https://maven.apache.org/guides/introduction/introduction-to-dependency-mechanism.html)。

## 七、传递依赖、排除与版本冲突

### 1. 什么是传递依赖？

如果项目 A 依赖 B，B 又依赖 C，Maven 可以读取 B 的 POM，继续解析 C。这样就不必手动声明每一个间接依赖。

但传递行为仍受作用域、可选依赖和排除规则影响，并不是无限制地引入所有库。

### 2. 如何排除依赖？

不需要某个直接依赖时，删除对应的 `<dependency>`。

如果要排除某条路径引入的间接依赖，可以使用 `<exclusions>`：

```xml
<dependency>
  <groupId>org.apache.commons</groupId>
  <artifactId>commons-text</artifactId>
  <version>1.12.0</version>
  <exclusions>
    <exclusion>
      <groupId>org.apache.commons</groupId>
      <artifactId>commons-lang3</artifactId>
    </exclusion>
  </exclusions>
</dependency>
```

这只演示排除语法，不是推荐的修复方案。如果库运行时需要被排除的类，可能报错。排除也只作用于这条依赖路径，其他路径仍可能引入同一个构件。

### 3. 同一个库出现不同版本怎么办？

Maven 通常优先选择依赖路径较近的版本；同深度时，先声明的优先。`dependencyManagement` 可以统一管理版本，但其中的条目本身不会自动引入依赖。

遇到问题时，可以先查看依赖树：

```cmd
mvn dependency:tree
```

查看“谁引入了谁”，比盲目删除依赖更有帮助。详细规则见 [官方依赖机制说明](https://maven.apache.org/guides/introduction/introduction-to-dependency-mechanism.html)。

## 八、Maven 生命周期

### 1. 三套独立的生命周期

| 生命周期 | 主要用途 | 常见阶段 |
| --- | --- | --- |
| `clean` | 清理构建产物 | `pre-clean` → `clean` → `post-clean` |
| `default` | 构建、验证和分发构件 | 编译、测试、打包、安装、部署等 |
| `site` | 生成项目文档站点 | `pre-site` → `site` → `post-site` → `site-deploy` |

`default` 中的主要阶段顺序如下，省略部分中间阶段：

```text
validate → compile → test → package
→ integration-test → verify → install → deploy
```

### 2. 常用命令

| 命令 | 作用 |
| --- | --- |
| `mvn clean` | 通常删除 `target` 等构建输出 |
| `mvn compile` | 编译正式代码 |
| `mvn test` | 执行到单元测试阶段 |
| `mvn package` | 执行前面的阶段并打包 |
| `mvn verify` | 执行到验证阶段，包含已配置的集成测试和检查 |
| `mvn install` | 将构件及 POM 等安装到本地仓库 |
| `mvn deploy` | 将构件发布到配置好的远程构件仓库，需要相应权限 |

调用一个阶段，会执行同一生命周期从开始到该阶段的流程。因此，`mvn package` 通常也会编译、测试，不必依次手动运行每个命令。

但 `clean` 属于另一套生命周期，不会被 `package` 自动调用。想先清理再打包，可以执行：

```cmd
mvn clean package
```

阶段中的具体工作由插件目标完成。没有绑定相应目标的阶段，不会凭空执行任务。Maven 的 `deploy` 是分发构件，也不等于将应用启动到服务器。参见 [Maven 生命周期说明](https://maven.apache.org/guides/introduction/introduction-to-the-lifecycle.html)。

## 九、测试有哪些分类？

### 1. 按测试范围和目标划分

| 类型 | 关注点 | 示例 |
| --- | --- | --- |
| 单元测试 | 较小代码单元的行为 | 验证一个计算方法 |
| 集成测试 | 多个组件之间的协作 | 服务能否向数据库保存记录 |
| 系统测试 | 完整系统是否满足要求 | 检查登录到下单的完整流程 |
| 验收测试 | 是否符合业务验收条件 | 按用户故事检查业务结果 |

### 2. 黑盒、白盒、灰盒测试

这是另一种划分角度：

- **黑盒测试**：主要依据输入、输出和外部规格设计用例。
- **白盒测试**：依据代码内部结构设计用例，例如分支和路径。
- **灰盒测试**：结合部分内部知识进行测试。

它们与测试范围不是一一对应的，例如系统测试可以采用黑盒方法。

### 3. Maven 和测试框架分别负责什么？

Maven 组织测试执行，测试框架表达测试用例。

常见配置中，Surefire 执行单元测试；集成测试通常使用 Failsafe，配置 `integration-test` 和 `verify` 目标，再通过 `mvn verify` 执行。仅写一个 `*IT.java` 文件，不代表它一定会自动运行。参见 [Failsafe 插件说明](https://maven.apache.org/surefire/maven-failsafe-plugin/)。

## 十、JUnit、断言与注解

JUnit 是 Java 测试框架。以下示例使用 JUnit Jupiter API。

### 1. 常用注解

| 注解 | 作用 |
| --- | --- |
| `@Test` | 标记测试方法 |
| `@BeforeEach` | 每个测试方法执行前运行 |
| `@AfterEach` | 每个测试方法执行后运行 |
| `@BeforeAll` | 测试类开始前运行，默认通常要求方法为 `static` |
| `@AfterAll` | 测试类结束后运行，默认通常要求方法为 `static` |
| `@DisplayName` | 设置便于阅读的测试名称 |
| `@Disabled` | 暂时禁用测试 |

不要混用 JUnit 4 的导入和 Jupiter 的导入。本文使用 `org.junit.jupiter.api`。

### 2. 断言是什么？

断言用来自动检查实际结果是否满足预期：

```java
assertEquals(3, actual);                      // 预期值在前，实际值在后
assertTrue(condition);                        // 检查条件
assertThrows(ArithmeticException.class, task); // 检查异常类型
```

只把结果打印出来，还需要人来判断；断言失败则会让测试失败，通常也会使构建失败。注解和断言的详细说明见 [JUnit 5.11.1 用户指南](https://docs.junit.org/5.11.1/user-guide/index.html)。

## 十一、动手练习：测试一个计算器

下面的代码应放在独立的 Java 练习项目中，不要覆盖 Hexo 博客项目的文件。这是待动手验证的示例。

### 1. 配置 pom.xml

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>
  <groupId>com.example</groupId>
  <artifactId>maven-demo</artifactId>
  <version>1.0-SNAPSHOT</version>

  <properties>
    <maven.compiler.release>17</maven.compiler.release>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
  </properties>

  <dependencies>
    <dependency>
      <groupId>org.junit.jupiter</groupId>
      <artifactId>junit-jupiter</artifactId>
      <version>5.11.1</version>
      <scope>test</scope>
    </dependency>
  </dependencies>

  <build>
    <plugins>
      <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-compiler-plugin</artifactId>
        <version>3.13.0</version>
      </plugin>
      <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-surefire-plugin</artifactId>
        <version>3.5.2</version>
      </plugin>
    </plugins>
  </build>
</project>
```

### 2. 编写正式代码

文件：`src/main/java/com/example/Calculator.java`。

```java
package com.example;

public class Calculator {
    public int add(int a, int b) {
        return a + b;
    }

    public int divide(int a, int b) {
        return a / b;
    }
}
```

### 3. 编写测试代码

文件：`src/test/java/com/example/CalculatorTest.java`。

```java
package com.example;

import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Test;

import static org.junit.jupiter.api.Assertions.*;

class CalculatorTest {
    private Calculator calculator;

    @BeforeEach
    void setUp() {
        calculator = new Calculator();
    }

    @Test
    @DisplayName("1 加 2 应等于 3")
    void addsNumbers() {
        assertEquals(3, calculator.add(1, 2));
    }

    @Test
    @DisplayName("整数除以零应抛出异常")
    void rejectsZeroDivisor() {
        assertThrows(ArithmeticException.class,
                () -> calculator.divide(1, 0));
    }
}
```

### 4. 运行与观察

在 Java 练习项目根目录打开 CMD，执行：

```cmd
mvn test
```

预期会发现并运行两个测试。报告通常位于 `target/surefire-reports`。

可以再把加法测试的预期值从 `3` 改成 `4`，观察测试和构建失败，再改回来。这样能直观理解断言如何帮助我们发现问题。

## 十二、下一步准备做什么？

接下来，我准备在一个小项目中完成依赖声明、查看依赖树、运行测试和打包，再分别观察本地仓库与 `target` 的变化。

复习时也可以试着回答：为什么 JUnit 使用 `test` 作用域？为什么 `package` 不会自动清理？`install` 和 `deploy` 分别把构件放到哪里？能结合实际操作解释这些问题，才算把笔记中的概念真正串了起来。
