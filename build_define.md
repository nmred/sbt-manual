## 配置文件 .sbt

### .sbt vs .scala 构建语句定义

一个项目的构建定义可以是在项目根目录中以 `.sbt` 后缀结尾的文件，也可以是一个在子目录 `project` 下以 `.scala` 结尾的文件

这章主要讨论 `.sbt` 文件定义，这种定义已经适合大部分情况. `.scala` 定义方式典型的用在多个 `.sbt` 文件分享共用的定义语句或者是复杂的项目构建中。更多信息参考[.scala 定义]()

### 什么是构建语句?

通过验证和解析构建语句文件， sbt 以一个不可变 Map（key-value 键值对）来描述构建过程结束

例如，一个 key 为 `name` 并且它的 Map 值是字符串，这个配置项`name`代表这个项目的名称

**构建语句定义不会直接修改影响 sbt 的Map**

相反，构建语句是一个由`Setting[T]`类型的对象构成大的列表构成，T是 Setting Map 值的类型， Setting可以通过以下几种方式修改map:添加一个新的键值对、修改以存在的key的值(在函数式编程中一个不可变数据结构和值，通过新赋值的方式来修改值，而不是在原值的基础上修改)

在build.sbt 中你可能要创建一个 `Setting[String]` 类型的配置项申明项目名称：

```
name := "hello"
```

这个 `Setting[String]` 通过添加或替换原有key值来修改，并且赋值为"hello" ，这个修改的 map 变成一个新的map。 创建一个 map，sbt 首先配置列表所有修改的配置放在一块，并且如果有的配置值依赖某些配置项将在依赖的配置项后处理。然后 sbt 利用排序后的配置项构建新的 map

总结：构建语句是一个 `Setting[T]` 组成的列表， Setting[T]是一个可以通过修改的map key-value 键值对构成， T是键值对值的类型

### 如何定义 build.sbt 配置项

build.sbt 是一个 Seq[Setting[_]]， 是一系列用空行分割 Scala 表达式，每行是这个序列的一个元素.

例如：

```
name := "hello"

version := "1.0"

scalaVersion := "2.10.3"

```

每个配置项都是一个 scala 表达式，在 build.sbt 中的表达式都是独立而不是一个 scala 代码块. 这些表达式由 val、lazy val 和 def 构成，对象和类不允许定义在 build.sbt 中，应该定义在子目录 `project` 中的 `.scala` 源代码文件中。

配置表达式的左值如 `name`, `version` 和 `scalaVersion` 是配置项的 key,  key 是 `SettingKey[T]`, `TaskKey[T]`或 `InputKey[T]` 的实例，其中 T 是期望值的类型，具体key的类型将在下面介绍。

Key 对象有一个方法 `:=` 调用将返回 `Setting[T]`, 你可以用 Java 语法风格调用：

```
name.:=("hello")
```

在 Scala 中允许 `name := "hello"` 方式调用（在Scala中对于单个参数的方法允许这种形式调用）

name 的 `:=` 方法返回一个 Setting 对象，其中具体的类型为Setting[String], 泛型类型 String 在 name key 本身定义中也出现了，但是 name 类型为 SettingKey[String]. 在这个例子中，将返回的 Setting[String] 对象通过添加或者替换得到一个新的 sbt 配置项 map， 并给定值为"hello"

如果给定一个错误类型的配置值，将无法编译通过

```
name := 42 // will not compile
```

#### 配置项之间必须用空行分割

不允许在budil.sbt将配置写成如下格式：

```
// will Not compile, not blank lines
name := "hello"
version := "1.0"
scalaVersion := "2.10.3"
```

sbt 需要一个分隔符用来判断一个表达式结束和另一个表达式起始， `.sbt` 文件中包含一系列 Scala 表达式，不是单个一个 Scala 项目，这些表达式必须分隔开并且单独进行编译.

### Keys

#### 类型

* SettingKey[T]: 这类key 的值只计算一次(当加载项目的时候计算完成后将一直保留)
* TaskKey[T]: 这类 key 的值将被作为任务调用，可以被重复调用计算的，也可能会产生影响
* InputKey[T]: 这类key 是针对一个任务需要传递一些输入参数

#### 内建 Keys

内建 Keys 是调用对象`Keys`的成员变量，对于 build.sbt 隐式包含 `import sbt.Keys._`, 所以`sbt.Keys.name`可以写作 `name`

#### 自定义 Keys

自定义 keys 可以分别用 `settingKey`, `taskKey`和`inputKey`方法创建. 每个方法定义期望关联值的类型和该key的描述信息， 每个key 的名称保存在 val 的常量中，例如， 定义一个名为 `hello` 的任务类型的key

```

lazy val hello = taskKey[Unit]("an example task")

```

.sbt 可以包含 vals 和 defs 的定义，所有这类型的定义将在解析配置项前执行， vals 和 defs 定义必须和所有配置项配置用空行分割

**注意：** 一般情况下推荐用lazy val 替换用 val 可以避免初始化值得顺序问题

#### 任务Keys 和 配置 Keys

TaskKey[T] 被称为一个任务，任务操作如 compile 或 package, 它们可能返回一个 Unit 类型(Unit 在 Scala 中相当于 void)或返回一个关联的任务，例如 package 这个任务将返回一个 TaskKey[File] 创建jar包的任务

当启动执行一个任务，例如执行 `compile` 在交互模式下， sbt 将执行和该任务相关的任务， sbt 项目描述表（map）中可以保存一个字符串的配置项（例如name配置项），也可以是保存可执行的代码块的任务（例如compile任务）， 即使一个可执行任务返回一个字符串，它也是在任何时候可重复执行的

#### 定义 task 和 settings

可以用 `:=` 给一个配置项或任务赋值，对于配置项的值将在项目加载的时候一次性计算，对于任务在执行的时候都会重新执行计算

例如：

```
// task
hello := {println("Hello!")}

// settings
name := "hello"
```

#### 任务和配置项的类型

Setting 通过一个任务key和一个配置项key创建出来的有细微的区别， `taskKey := 42` 结果是 `Setting[Task[T]]` 然而 `settingKey := 42` 结果是 `Setting[T]`. 对于大部分情况下基本看出来区别，因为任务key依然创建一个类型为T的值在任务执行的时候

T 和 Task[T] 不同的另一个含义： 一个配置项不能依赖一个任务，因为配置项仅在项目初始化的时候计算一次.

#### Keys 在 sbt 的交互模式

在sbt交互模式下，你可以执行任何任务key,当输入compile的时候将执行 任务key为compile的任务，如果给定的key类型不是任务而是一个配置项时将输出配置项的值，如果key为任务类型的执行结果将不会输出到终端，要看任务的输出结果需要执行 `show <task name>` 而不是`<task name>` 。习惯性的定义 sbt 的key的时候使用和Scala命名风格一样的驼峰命名。

了解一个key的更多信息，可以在交互模式下使用 `inspect <keyname>` ， 可以看到该key的值的类型和摘要描述信息等

#### 在 build.sbt 中导入包

你可以再 build.sbt 顶部使用 `import` 语句， 它们不需要用空行分割。sbt 默认情况下隐式的导入了以下包：

```
import sbt._
import Process._
import Keys._
```

#### 添加依赖包

要添加一个第三方库的依赖有两种方法，一种是直接将jar包放到 lib目录下，另一种方法是在build.sbt中添加依赖配置，如以下的方法：

```
libraryDependencies += "org.apache.derby" % "derby" % "10.4.1.3"
```

以上这个配置将在项目中添加一个 Apache Derby的库，并且版本为 10.4.1.3

libraryDependencies key 包含两个方法：`+=`(不是`:=`)和`%`， += 是在原来值上追加新值而不是替换原值，更多的解释可参考[配置配置项](). `%`方法作用是构建一个 Ivy 模块ID在依赖库中被解析。
