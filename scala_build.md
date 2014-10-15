# 配置文件 .scala

### sbt的递归性

build.sbt 是非常简单的，其隐藏了sbt真正工作的一些细节，sbt 是由Scala语言编写的，其自身也需要构建，那么由什么好的办法来实现呢？

`project`目录是在构建项目中的另一个项目，它负责整个项目的构建定义，理论上在`project`目录下还可以有另一个`project`项目(递归)，其构建的是sbt项目本身用来支撑上级项目的构建。

例如，你可以在构建项目下再次创建一个项目，以下是目录层级结构：

```
hello/                  # your project's base directory

    Hello.scala         # a source file in your project (could be in
                        #   src/main/scala too)

    build.sbt           # build.sbt is part of the source code for the
                        #   build definition project inside project/

    project/            # base directory of the build definition project

        Build.scala     # a source file in the project/ project,
                        #   that is, a source file in the build definition

        build.sbt       # this is part of a build definition for a project
                        #   in project/project ; build definition's build
                        #   definition


        project/        # base directory of the build definition project
                        #   for the build definition

            Build.scala # source file in the project/project/ project
```
不用担心，大部分情况下是不需要创建这个的，但是理解这个概念对运用sbt很有帮助。

顺便说一下，任何以 `.sbt`或`.scala`后缀的定义文件都会被用到，经常说的build.sbt或Build.scala命名只是为了方便而已，也就是说sbt配置支持多文件配置。

### .scala 配置文件定义

.sbt文件定义将被合并到子目录`project`中，例如如下项目目录结构：

```
hello/                  # your project's base directory

    build.sbt           # build.sbt is part of the source code for the
                        #   build definition project inside project/

    project/            # base directory of the build definition project

        Build.scala     # a source file in the project/ project,
                        #   that is, a source file in the build definition
```

在build.sbt中的 Scala 配置表达式将会被编译合并到Build.scala中（或者是在project目录下的任何`.scala`文件中）。
在项目根目录的`*.sbt` 文件将会变成在根目录下`project`构建项目定义的一部分。`.sbt`只是为了定义项目方便。

### build.sbt 和 Build.scala 的关系

对于在构建项目定义中混合的 `.sbt`和`.scala`定义，你需要理解它们之间的关系，以下是两个文件的例子，首先，假设一个项目 hello， 创建`hello/project/Build.scala`如下：

```
import sbt._
import Keys._

object HelloBuild extends Build {
  val sampleKeyA = settingKey[String]("demo key A")
  val sampleKeyB = settingKey[String]("demo key B")
  val sampleKeyC = settingKey[String]("demo key C")
  val sampleKeyD = settingKey[String]("demo key D")

  override lazy val settings = super.settings ++
    Seq(
      sampleKeyA := "A: in Build.settings in Build.scala",
      resolvers := Seq()
    )

  lazy val root = Project(id = "hello",
    base = file("."),
    settings = Seq(
      sampleKeyB := "B: in the root project settings in Build.scala"
    ))
}
```

创建`hello/build.sbt`如下：

```
sampleKeyC in ThisBuild := "C: in build.sbt scoped to ThisBuild"

sampleKeyD := "D: in build.sbt"
```

启动sbt的交互模式，输入`inspect sampleKeyA`将会看到：

```
[info] Setting: java.lang.String = A: in Build.settings in Build.scala
[info] Provided by:
[info]  {file:/home/hp/checkout/hello/}/*:sampleKeyA
```

然后输入` inspect sampleKeyC` 显示如下：

```
[info] Setting: java.lang.String = C: in build.sbt scoped to ThisBuild
[info] Provided by:
[info]  {file:/home/hp/checkout/hello/}/*:sampleKeyC
```

"Provided by" 显示这两个参数配置的作用域是相同的，在 `.sbt`文件中配置`sampleKeyC in ThisBuild `等同于在 `.scala`文件中的Build.settings 中配置的值，在上述两个地方配置的值得作用域都是工程级别的作用域。

现在，在查看sampleKeyB：

```
[info] Setting: java.lang.String = B: in the root project settings in Build.scala
[info] Provided by:
[info]  {file:/home/hp/checkout/hello/}hello/*:sampleKeyB
```

sampleKeyB 的作用域是项目维度的(`{file:/home/hp/checkout/hello/}hello`)不再是工程级别的作用域。

你可能猜到`inspect sampleKeyD` 和sampleKeyB的一样：

```
[info] Setting: java.lang.String = D: in build.sbt
[info] Provided by:
[info]  {file:/home/hp/checkout/hello/}hello/*:sampleKeyD
```

sbt 在 `.sbt` 文件中追加配置Build.settings 和 Project.settings 配置的优先级比`.scala`文件的高，所以配置在 `.sbt` 文件中的sampleC 或 sampleD会修改Build.scala的配置。

另一个需要注意的是：sampleKeyC 和 sampleKeyD 可以定义在build.sbt 中定义，这是因为sbt会将Build 对象自动隐式的导入到`.sbt`文件中，例如这个例子sbt 会将 `HelloBuild._ ` 隐式的导入到build.sbt 文件中。

总结：

* 在`.scala`文件中，可以定义 Build.settings f配置项供sbt查找，其作用域自动为工程级别的作用域
* 在`.scala`文件中，可以定义 Project.settings配置项供sbt查找，其作用域自动为项目维度的作用域
* 在`.scala`文件中的任何Build对象，都会自动的导入到所有的`.sbt`文件中
* `.sbt` 文件中的配置将会被追加到`.scala` 文件中
* 配置在`.sbt`文件中的配置默认是项目维度的作用域，除非手动指定其他作用域

### 什么时候用到 .scala 配置文件

`.scala` 配置文件可以用任何的Scala代码编写，包括顶级的类和对象，由于它是标准的Scala语法，所以也没有必须添加空行来分割配置的限制

在配置参数配置推荐用`.sbt`配置文件，当要实现一些任务配置或者要定义一些复用配置供多个`.sbt`文件使用的情况推荐使用`.scala`配置文件

### 在交互模式下构建项目

你可以在交互模式下，切换到在`project`目录下的构建工程项目的项目中, 当切换到该项目中后可以执行一些操作，如`reload plugins.`等

```
> reload plugins
[info] Set current project to default-a0e8e4 (in build file:/home/hp/checkout/hello/project/)
> show sources
[info] ArrayBuffer(/home/hp/checkout/hello/project/Build.scala)
> reload return
[info] Loading project definition from /home/hp/checkout/hello/project
[info] Set current project to hello (in build file:/home/hp/checkout/hello/)
> show sources
[info] ArrayBuffer(/home/hp/checkout/hello/hw.scala)
>
```

正如上面的例子，你可以用`reload return `命令来退出当前工程构建项目的项目，回到常规的项目中。

### 不可变配置

sbt 配置可能被错误的理解为在build.sbt的配置将会被添加到Build 或Project 对象的settings 字段中，其实settings 是 Build 和 Project的列表，build.sbt中的配置将会被和一个不可变的配置列表连接起来生成一个新的列表供sbt使用的， Build 和 Project中的“不可变配置”列表仅仅是完成sbt配置的一部分。


事实上，还有其他的配置文件，它们将会按照如下顺序添加：

* 配置在`.scala`文件中的Build.settings和Project.settings配置项
* 用户全局配置，例如在` ~/.sbt/0.13/global.sbt`文件中可以配置影响所有工程构建的配置项
* 通过被插件注入的配置项
* 项目中`.sbt`文件配置的配置项
* 构建工程项目的项目(例如：所有项目中的project项目)从全局插件中添加的配置项

后面的配置将会重载前面的配置，最后生成一个配置列表。
