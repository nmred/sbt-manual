# 目录结构

### 根目录

在 sbt 术语中 “根目录”是一个包含项目的目录，所以如果创建一个 hello 项目将包含 hello/build.sbt 和 hello/hw.scala 在 hello world 项目例子中，其中hello是根目录

### 源代码目录结构

源代码可以放到项目的根目录类似于 hello/hw.scala , 但是在真正的项目很少利用这样的代码结构，这样会使项目变得混乱， sbt 的项目目录结构默认情况下和 Maven 一样（所有路劲是基于根目录的相对路劲）:

```
src/
    main/
        resources/
            <包含在main 的jar包中的文件>
        scala/
            <scala源代码>
        java/
            <java 源代码>
    test/
        resources/
            <包含在test 的jar包中的文件>
        scala/
            <scala 源代码>
        java/
            <java 源代码>

```

除 src/ 目录以外的目录将被忽略，包括隐藏的目录。

### sbt 构建定义文件

你已经在项目的根目录中看到了 build.sbt ， 其他的 sbt 定义文件在子目录 project 中， `project` 可以包含 `.scala` 文件，将和 `.sbt` 定义进行合并来完成构建定义，详细的可以参考 [.scala 配置定义]()

```
    build.sbt
    project/
        Build.scala
```

你可能看到在 `project/` 目录中有一个 `.sbt` 文件，这个文件和根目录中的 `.sbt` 不是针对一个项目的定义，稍后会解释这一点

### 构建项目

生成的文件(编译后的 class文件，jar 包，项目管理文件，缓存文件和文档)将被写入到一个`target` 目录默认

### 配置项目版本控制

项目的 `.gitignore` 文件中应该包含 `target/` , 注意：以 `/` 结尾（匹配目录中所有目录和文件）并且开头不包含 `/` （为了匹配 project/target/）
