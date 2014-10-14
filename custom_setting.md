# 自定义配置和任务

### 定义一个配置

在[配置文件 .sbt](build_define.html)这章已经将了如何定义个配置，大部分配置定义在[Default](http://www.scala-sbt.org/0.13/sxr/sbt/Defaults.scala.html)中

配置有三种类型，其中SettingKey和TaskKey已经在[配置文件 .sbt](build_define.html)介绍了，InputKey在任务配置的输入章节介绍。

对于配置的一些例子：

```
val scalaVersion = settingKey[String]("The version of Scala used for building.")
val clean = taskKey[Unit]("Deletes files produced by the build, such as generated sources, compiled classes, and task caches.")
```

创建配置的构造方法有两个参数，分别是配置的名称("scalaVersion")和一个描述该配置的字符串("The version of scala used for building.")。

在[配置文件 .sbt](build_define.html)中介绍在SettingKey[T]中的T表示该配置值得类型，在TaskKey[T]中T代表该任务返回的类型。并且也介绍了一个参数配置是一个常量，其在项目加载的时候就初始化好了，而一个任务配置是可以重复执行的（任何时候在交互模式下或批量脚本中都可以调用执行）

所有的配置都定义在 `.sbt`, `.scala`文件或插件中，任何的在`.scala`配置文件中或在插件中的配置都将自动的合并到 `.sbt` 文件中。

### 实现一个任务

当定义完一个任务配置后，需要实现一个任务配置。可以自己实现一个任务，也可以重载一个已经存在的任务，两者配置的方式没有任何区别。通过`:=`方法来关联任务配置来实现一个任务，例如：


```
val sampleStringTask = taskKey[String]("A sample string task.")

val sampleIntTask = taskKey[Int]("A sample int task.")

sampleStringTask := System.getProperty("user.home")

sampleIntTask := {
  val sum = 1 + 2
  println("sum: " + sum)
  sum
}
```

如果一个任务有依赖关系，可以直接引用依赖的配置，在[配置参数的方法](kind_setting.html) 一章已经介绍了。

实现任务最难的部分是用Scala代码实现该任务具体执行过程，例如，可以编写一个格式化HTML的任务用相关的HTML lib 库（自己可以定义添加一个依赖库，在该基础上编写)

sbt 也提供了一些工具库，例如常用的文件和目录操作API等。

### 利用插件

如果有许多通用的代码，可以将其提取到一个插件中，可以实现多项目的复用。
