## Maven

#### 概述

Maven 是一个项目管理和综合工具。Maven 提供了开发人员构建一个完整的生命周期框架。开发团队可以自动完成项目的基础工具建设，Maven 使用标准的目录结构和默认构建生命周期。

在多个开发团队环境时，Maven 可以设置按标准在非常短的时间里完成配置工作。由于大部分项目的设置都很简单，并且可重复使用，Maven 让开发人员的工作更轻松，同时创建报表，检查，构建和测试自动化设置。

Maven 提供了开发人员的方式来管理：

- Builds（构建）
- Documentation（文档管理）
- Reporting（报告）
- Dependencies（依赖）
- SCMs
- Releases
- Distribution
- mailing list

概括地说，Maven 简化和标准化项目建设过程。处理编译，分配，文档，团队协作和其他任务的无缝连接。 Maven 增加可重用性并负责建立相关的任务。

#### Maven的本地仓库

Maven 的本地资源库是用来存储所有项目的依赖关系(插件 Jar 和其他文件，这些文件被 Maven 下载)到本地文件夹。很简单，当你建立一个 Maven 项目，所有相关文件将被存储在你的 Maven 本地仓库。

默认情况下，Maven 的本地资源库默认为 `.m2` 目录文件夹：

通常情况下，可改变默认的 `.m2` 目录下的默认本地存储库文件夹到其他更有意义的名称，例如， maven-repo 找到 `{M2_HOME}\conf\setting.xml`, 更新 `localRepository` 到其它名称。



当你建立一个 Maven 的项目，Maven 会检查你的 `pom.xml` 文件，以确定哪些依赖下载。首先，Maven 将从本地资源库获得 Maven 的本地资源库依赖资源，如果没有找到，然后把它会从默认的 Maven 中央存储库 http://repo1.maven.org/maven2/ 查找下载。

使用 MVNrepository 搜索：https://mvnrepository.com/



#### Maven POM

POM 代表项目对象模型。它是 Maven 中工作的基本单位，这是一个 XML 文件。它始终保存在该项目基本目录中的 pom.xml 文件。

POM 包含的项目是使用 Maven 来构建的，它用来包含各种配置信息。

POM 也包含了目标和插件。在执行任务或目标时，Maven 会使用当前目录中的 POM。它读取POM得到所需要的配置信息，然后执行目标。部分的配置可以在 POM 使用如下：

- project dependencies
- plugins
- goals
- build profiles
- project version
- developers
- mailing list

创建一个POM之前，应该要先决定项目组(groupId)，它的名字(artifactId)和版本，因为这些属性在项目仓库是唯一标识的。

**例子：** 

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
   xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
   xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
   http://maven.apache.org/xsd/maven-4.0.0.xsd">
    
   <modelVersion>4.0.0</modelVersion>
   <groupId>com.test.product</groupId>
   <artifactId>project</artifactId>
   <version>1.0</version>
    
<project>
```

要注意的是，每个项目只有一个 POM 文件

- 所有的 POM 文件在项目元素必须有三个必填字段: groupId，artifactId，version
- 在库中的项目符号是：`groupId:artifactId:version`
- `pom.xml` 的根元素是 project，它有三个主要的子节点。

| 节点       | 描述                                                         |
| ---------- | ------------------------------------------------------------ |
| groupId    | 这是项目组的编号，这在组织或项目中通常是独一无二的。 例如，一家银行集团 `com.company.bank` 拥有所有银行相关项目。 |
| artifactId | 这是项目的 ID。这通常是项目的名称。 例如，`consumer-banking`。 除了 groupId 之外，artifactId 还定义了 artifact 在存储库中的位置。 |
| version    | 这是项目的版本。与 groupId 一起使用，artifact 在存储库中用于将版本彼此分离。 例如：`com.company.bank:consumer-banking:1.0`，`com.company.bank:consumer-banking:1.1` |

