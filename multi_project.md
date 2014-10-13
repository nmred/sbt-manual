# 多项目构建

### 多项目

通常在一个工程中构建多个项目间会有关联，尤其是它们都依赖一个项目时可以很容易的更新项目

在一个工程中每个子项目都会有自己的源代码目录、生成各自的jar包当执行 `package` 时.

一个项目通过申明一个 `Project` 类型的懒值来定义，例如：

```
lazy val util = project

lazy val core = project
```

这个变量值名称将被用来当做 `Project Id` 和项目的根目录名称，这个ID将用来在命令行中引用项目，利用方法`in` 可以修改默认的项目根目录。例如， 以下是更加明确的申明项目：

```
lazy val util = project.in(file("util"))

lazy val core = project in file("core")
```

### 项目间依赖

在一个工程中一个项目完全可能依赖另一个项目，经常有两种依赖方式：聚合和classpath

#### 项目聚合

聚合的意思是当运行一个任务在一个项目中，其通过聚合方式依赖的项目也会执行，例如：

```
lazy val root = (project in file(".")).
  aggregate(util, core)

lazy val util = project

lazy val core = project
```

在上面的例子中，`root`项目聚合了项目`util`和`core`，编译`root`项目将看到三个项目被编译。

在项目聚合过程中，以`root`项目为例，是可以控制任务维度作用域的配置的，例如,可以控制在执行`update`这个任务时不进行聚合：

```
lazy val root = (project in file(".")).
  aggregate(util, core).
  settings(
    aggregate in update := false
  )

[...]
```

`aggregate in update ` 表示在`update`这个任务维度作用域中aggregate的配置。（具体可参考[作用域](scope.html)章节）

> 注意：在聚合过程中聚合是并行处理的，所以被聚合的项目时没有先后顺序的

#### classpath 依赖

在源代码层级一个项目可能会依赖另一个项目，可以通过 `dependsOn` 方法添加依赖关系，例如，`core`项目需要在classpath中指定`util`项目，可以这样定义`core`项目:

```
lazy val core = project.dependsOn(util)
```

这样就可以在`core`项目中调用`util`项目的方法， 当编译的时候会有编译顺序，`util`必须在`core`之前编译， 如果依赖多个项目，可以给`dependsOn`方法指定多个参数，例如，`dependsOn(bar, baz)`

#### 配置维度作用域的 classpath 依赖

`foo dependsOn(bar)` 表示foo在`Compile`这个配置维度作用域下依赖在配置维度作用域为`Compile`下的`bar`项目，确切的写法应该为：`dependsOn(bar % "compile->compile")`, 在"compile->compile"中的`->`表示项目间的依赖关系，所以"test->compile"可以表示为在`Test`配置维度作用域下的foo依赖`Compile`配置维度作用域下的bar

省略"->config"这部分隐含意思为"->compile", 所以`dependsOn(bar % "test") `意思是在`Test`配置维度作用域下的foo依赖`Compile`配置维度作用域下的bar

可以声明为"test->test"，意思为在`Test`配置维度作用域下项目依赖，例如，在`src/test/scala`目录的公共类库可以在`src/test/scala`源代码中使用。

针对一个依赖关系可以配置多个配置维度的作用域，用分号分隔，例如：`dependsOn(bar % "test->test;compile->compile")`

### 默认根项目

如果一个项目没有在工程根目录下定义，sbt将创建一个聚合整个工程子项目的默认项目。

由于项目hello-foo 定义了`base = file("foo")`,项目目录为子目录foo， 源代码可以直接放到foo目录中，类似`foo/Foo.scala`或放到`src/main/scala`,如果是采用sbt工程构建标准目录，foo目录下还包括工程构建定义文件。

在foo目录下的任何`.sbt`文件将被合并在一起，并且定义的配置项属于项目维度`hello-foo`作用域。

在hello工程中允许子项目配置不同的版本，可以在配置文件:`hello/build.sbt, hello/foo/build.sbt, 和hello/bar/build.sbt`中配置不同版本，现在在交互模式下执行`show version`将看到如下信息：

```
> show version
[info] hello-foo/*:version
[info]  0.7
[info] hello-bar/*:version
[info]  0.9
[info] hello/*:version
[info]  0.5
```

`hello-foo/*:version`被定义在`hello/foo/build.sbt`中 `hello-bar/*:version`被定义在`hello/bar/build.sbt`中 `hello/*:version`被定义在`hello/build.sbt`中

每个version是不同的项目维度作用域，但是这三个build.sbt部分配置是相同的

> 每个项目的配置都在自己项目目录下的 .sbt 文件中配置，其实对于上述配置有更简单的方法，那就是配置到 .scala 文件中，列举出项目和对应的项目根目录

你会发现将多个项目按顺序定义在 .scala 配置文件中会比单独定义在各自的目录下更加清晰，不过这个自己决定。

不允许定义一个子目录为`project`的目录，`foo/project/Build.scala`将会被忽略

### 交互模式下操作项目

在sbt的交互模式下，使用命令`projects` 列举出该工程的所有项目，命令` project <projectname>`可以切换当前项目。当运行一个任务的时候（比如`compile`）将在当前项目下运行，所以没有必要在根项目中编译，可以单独编译一个子项目。

### 配置代码复用

.sbt 配置文件中的定义在多个 .sbt 中是相互不可见的，为了是多个.sbt间复用配置，可以在根目录的project目录下定义一个或多个 .scala 的文件，这个目录其实也是一个sbt工程，只不过这个的作用是工程构建。

例如：

`<root>/project/Common.scala:`
```
import sbt._
import Keys._

object Common {
  def text = "org.example"
}
```

在.sbt中可以直接调用：

`<root>/build.sbt:`
```
organization := Common.text
```
