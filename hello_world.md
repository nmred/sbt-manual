# 简单例子：Hello World

### 创建项目目录和项目代码

一个合法的 sbt 项目可以在一个项目目录中包含单个文件。尝试创建一个包含hw.scala 文件的目录hello, 文件中的内容如下：

```
object Hi {
    def main(args: Array[String]) = println("Hi!")
}
```

现在可以进入目录 hello 运行 sbt 命令， 在sbt交互模式下运行run 命令， 具体的在 Unix或 OS X 中的命令如下：

```
$ mkdir hello
$ cd hello
$ echo 'object Hi { def main(args: Array[String]) = println("Hi!") }' > hw.scala
$ sbt
...
> run
...
Hi!
```

在这种情况下 sbt 完全遵循一套构建规则的，sbt 会自动根据规则进行构建，具体的规则如下：

* 代码源文件可以是 sbt 项目根目录
* 代码源文件可以是 在 src/main/scala 或 src/main/java 目录
* 测试代码目录为 src/test/scala 或 src/test/java 目录
* 数据文件在 src/main/resources 或 src/test/resources
* 依赖的 jars 文件可以放到 lib 目录下

默认情况下 sbt 构建的项目用的 scala 版本和 sbt 自身运行的scala版本是一样的，可以通过运行 sbt run 命令或 sbt console 进入 Scala REPL 模式下运行项目， sbt 会加载依赖的 classpath ，所以可以使用 sbt 直接运行测试项目。

### 构建项目的配置文件

许多项目都需要手动进行配置，最基本的配置一般都是定义在根目录的 build.sbt 文件中， 例如， 如果项目跟目录为 hello ， 在 hello/build.sbt 中可能为：

```
name := "hello"

version := "1.0"

scalaVersion := "2.10.3"

```

需要注意的是每个配置项之间用空行分割，这个不仅仅是为了显示，实际上 sbt 需要根据空行来分割多个配置项的。在[配置文件 .sbt](build_define.html) 章节中你可以学到如何配置 build.sbt

如果你需要将项目打包成 jar 包，需要在 build.sbt 中指定名称和最新版本号。

### 设置 SBT 版本

可以强制使用某个 sbt 版本在构建项目的时候，需要在 hello/project/build.properties 文件中配置：

```
sbt.version = 0.13.5
```

强制使用 sbt 的 0.13.5 版本， 虽然sbt 版本间 99% 是兼容的，不过设置hello/project/build.properties 指定 sbt 版本可以避免版本之间不兼容导致的一些潜在问题。
