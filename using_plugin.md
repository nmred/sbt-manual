# 插件使用

### 什么是插件？

插件可以扩展工程构建定义，通常是增加一些新的配置，配置可以是任务配置，例如，一个插件可以增加一个`codeCoverage` 的任务配置，用来生成项目单元测试的代码覆盖率报告。

### 使用插件

如果一个项目的目录为hello, 并且在该项目中使用`sbt-site`这个插件，只需创建一个名为`hello/project/site.sbt`配置文件，并且通过`addSbtPlugin`方法申明该插件的Ivy模块ID：

```
addSbtPlugin("com.typesafe.sbt" % "sbt-site" % "0.7.0")
```

如果添加一个`sbt-assembly`的插件，创建一个` hello/project/assembly.sbt` 配置文件，并且配置如下：

```
addSbtPlugin("com.eed3si9n" % "sbt-assembly" % "0.11.2")
```

并不是每个插件在默认的远程库中都存在，这个需要根据具体的插件文档来配置远程库地址：

```
resolvers += Resolver.sonatypeRepo("public")
```

插件一般都会提供一些配置来添加启用插件，下一小节讨论这个问题。


### 插件的启用

一个插件通过配置可以自动的添加到项目中，不需要额外的做任何工作，在0.13.5有一个新的特性是可以自动启用和配置在一个项目中，大部分自动插件都已经自动的进行了默认的配置，不过有个别的插件也需要明确启用。

如果一个插件需要配置启用才可以使用，需要在build.sbt中进行如下配置：

```
lazy val util = (project in file("util")).
  enablePlugins(FooPlugin, BarPlugin).
  settings(
    name := "hello-util"
  )
```

`enablePlugins` 方法用来指定需要启用的配置。

一个项目也可以通过`disablePlugins` 来排除使用一个插件。例如，在util项目中不想用插件`IvyPlugin`，可以如下配置：

```
lazy val util = (project in file("util")).
  enablePlugins(FooPlugin, BarPlugin).
  disablePlugins(plugins.IvyPlugin).
  settings(
    name := "hello-util"
  )
```

自动插件应该在文档中明确指定该插件是否需要显式的启用， 如果疑惑一个插件是否已经启用，可以在交互模式下使用命令`plugins`来判断,例如：
```
> plugins
In file:/home/jsuereth/projects/sbt/test-ivy-issues/
        sbt.plugins.IvyPlugin: enabled in scala-sbt-org
        sbt.plugins.JvmPlugin: enabled in scala-sbt-org
        sbt.plugins.CorePlugin: enabled in scala-sbt-org
        sbt.plugins.JUnitXmlReportPlugin: enabled in scala-sbt-org
```

这块显示的是所有默认开启的插件，sbt 默认启用三个插件：

* CorePlugin: 提供并行任务的控制
* IvyPlugin： 提供依赖模块的发布、解析机制
* JvmPlugin： 提供编译、测试、打包 Scala/Java代码机制

另外 `JUnitXmlReportPlugin `插件只要是提供生成 junit-xml 文件的支持

添加老版非自动插件经常需要显式的配置，插件文档将会指出如何进行配置，典型的老版插件是由基础配置和自定义配置构成。

例如，对于 `sbt-site` 这个插件，用如下配置来启用该插件：

```
site.settings
```

如果是多项目构建，可以直接在项目配置中配置：

```
// don't use the site plugin for the `util` project
lazy val util = (project in file("util"))

// enable the site plugin for the `core` project
lazy val core = (project in file("core")).
  settings(site.settings : _*)
```

### 全局插件

插件可以通过在` ~/.sbt/0.13/plugins/` 下一次申明配置应用在多个项目中，` ~/.sbt/0.13/plugins/` 是一个sbt项目，其classpath将导出到每个sbt构建项目中，粗略的讲，在` ~/.sbt/0.13/plugins/`中的任何`.sbt`或`.scala`配置都会影响到所有的sbt构建项目
可以在`~/.sbt/0.13/plugins/build.sbt` 通过`addSbtPlugin`方法添加一个插件来应用到所有的项目中，所以如果添加一些针对机器的环境变量等的场合这个特性非常实用。

### 已经存在的插件

已经存在的[插件列表](http://www.scala-sbt.org/0.13/docs/Community-Plugins.html)

一般比较常用的有两类：

* 针对某些IDE的插件
* 针对Web框架支持的插件，如[xsbt-web-plugin](https://github.com/JamesEarlDouglas/xsbt-web-plugin)
