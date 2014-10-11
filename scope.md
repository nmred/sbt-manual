# 作用域

### 深入 Keys

前面我们一直简单的认为类似于 `name` 的 key 将一一对应一个值，在 sbt 中其实就是一个 key-value 的map 表，事实上每个 key 除了关联一个值外还有一个上下文关系，被称为“作用域”

例如：

*  如果在一个项目构建定义中含有多个项目，那么一个 key 可能有不同的值在不同的项目中
*  对于项目代码和项目测试代码中 `compile` 这个key对应的值是不同的
*  `packageOptions` （包含打包jar文件的所有参数）key 在打包 class 二进制文件和源代码文件时的值是不同的

对于 `name` 这个key不是只有一个值，值会随作用域的不同而不同，但是在同一个作用域中一个key只有一个值

以前认为 sbt 是通过处理一个配置列表生成 key-value 的一个 map表来描述整个项目构建的，现在可以把这个map认为是一个由作用域的 map表，在项目构建定义中（如build.sbt文件中定义) 每个 key 都有一个对应的作用域。

一般情况下每个 key 都有一个隐含或默认的作用域，但是如果默认的不是想要的需要显式覆盖声明

### 作用域的维度

每个作用域维度是一个类型，每个类型实例都可以定义自己唯一值得key, 以下是三种作用域维度：

* 项目维度
* 配置维度
* 任务维度

#### 作用域的项目维度

如果在一个构建工程中定义多个项目，每个项目拥有自己唯一的配置，那么这时key的作用域是一个具有项目维度的作用域。

作用域项目维度可以设置为“整个工程”有效（可以称为该作用域为工程作用域），这样一个配置将作用于整个构建工程中而不是单一的一个项目，构建级别的配置经常用来当做备用，当某个项目中没有配置该配置的时候。

#### 作用域的配置维度

一个配置维度定义一个构建类型，可能有自己的 `classpath`、 源代码目录、打包发布等，配置维度这个概念来源于 Ivy, 由于 Sbt 的包依赖管理用的是 MavenScopes

在 sbt 中的一些配置维度：

* Compile : 定义编译项目配置 (src/main/scala)
* Test : 定义测试项目的配置(src/test/scala)
* Runtime : 定义运行一个工程时的配置

默认情况下，在编译、打包、运行是所有的key将对应关联一个配置维度，所以在不同的配置维度下运行结果可能不同。最常见如任务类型的key `run`, `compile`, `package`, 其实所有的key都会受配置维度的作用域的影响，例如`sourceDirectories `, `scalacOptions`,`fullClasspath`等

#### 作用域的任务维度

配置可以影响一个任务的执行，例如， `packageSrc` 会受到配置参数`packageOption`的影响。为了实现这个功能在 sbt 中`packageSrc`可以当做配置参数 `packageOption`的作用域

打包构建有多个任务(packageSrc, packageBin, packageDoc)可以共享和打包相关的配置参数，例如 `artifactName`和`packageOptions`, 但是他们的值在不同的任务维度下是不同。

### 全局作用域

每个作用域维度都是由一个维度类型实例构成(例如任务维度就是由一个任务实例构成)， 一个维度也可以由一个全局值构成

全局的概念正如你所理解的，一个参数配置值将被应用到所有的维度实例中，例如一个任务维度是全局的，那么这个配置将在所有任务中有效。

### 委托

当一个作用域中没有定义某个 key, 那么其在该作用域是没有关联的值。对于每个作用域，sbt 通过搜索其他作用域的路劲作为某个key 的备选作用域，典型的例子：如果一个key在指定作用域中没有关联的值，sbt试图从其他作用域获取一个值，例如全局作用域或者工程作用域。

这个特性允许你在一个作用域中设置某个key，在其他的作用域中继承该key, 可以使用sbt的inspect命令查看某个key的搜索作用域详细信息。

### 作用域在运行sbt时相关解释

在交互模式下或命令下， sbt 将用下面的形式表示作用域：

```
{<build-uri>}<project-id>/config:intask::key
```

* `{<build-uri>}/<project-id>` 表示一个项目维度的作用域，当表示整个工程的作用域时 `<project-id>` 部分可以省略
* config 表示作用域的配置维度
* intask 表示作用域的任务维度
* key 表示配置的key

> `*`在上述任意段中出现表示在该作用域维度下是一个全局作用域

如果key省略指定某一部分，将按以下规则推断作用域：

* 如果省略项目维度将被认为当前的项目
* 如果一个依赖配置维度的key在省略配置或任务维度时将自动探测

### 作用域的例子详解

* `*:fullClasspath ` 指定一个全局的配置，而不是默认配置
* `doc::fullClasspath` 配置在doc任务中fullClasspath参数，项目和配置维度默认
* `{file:/home/hp/checkout/hello/}default-aea33a/test:fullClasspath` 表示在工程{file:/home/hp/checkout/hello/}中的项目default-aea33a下的配置维度为test的fullClasspath配置参数，任务维度为默认的
* `{file:/home/hp/checkout/hello/}/test:fullClasspath` 表示是一个工程级别的作用域，工程为{file:/home/hp/checkout/hello/}
* `{.}/test:fullClasspath` 表示一个工程级别的作用域，这块的 `{.}.{.}` 在Scala代码中可以写成 ThisBuild
* `{file:/home/hp/checkout/hello/}/compile:doc::fullClasspath ` 表示 fullClasspath 配置参数设置所有的作用域维度

### 检测作用域

在 sbt 交互模式下可以使用命令`inspect` 来检测一个配置参数的作用，例如

```
> inspect test:fullClasspath
```

命令执行返回结果如下：

```
[info] Task: scala.collection.Seq[sbt.Attributed[java.io.File]]
[info] Description:
[info]  The exported classpath, consisting of build products and unmanaged and managed, internal and external dependencies.
[info] Provided by:
[info]  {file:/home/hp/checkout/hello/}default-aea33a/test:fullClasspath
[info] Dependencies:
[info]  test:exportedProducts
[info]  test:dependencyClasspath
[info] Reverse dependencies:
[info]  test:runMain
[info]  test:run
[info]  test:testLoader
[info]  test:console
[info] Delegates:
[info]  test:fullClasspath
[info]  runtime:fullClasspath
[info]  compile:fullClasspath
[info]  *:fullClasspath
[info]  {.}/test:fullClasspath
[info]  {.}/runtime:fullClasspath
[info]  {.}/compile:fullClasspath
[info]  {.}/*:fullClasspath
[info]  */test:fullClasspath
[info]  */runtime:fullClasspath
[info]  */compile:fullClasspath
[info]  */*:fullClasspath
[info] Related:
[info]  compile:fullClasspath
[info]  compile:fullClasspath(for doc)
[info]  test:fullClasspath(for doc)
[info]  runtime:fullClasspath
```

在第一行中可以看到一个任务key, 其值得类型是 `scala.collection.Seq[sbt.Attributed[java.io.File]].`

"Provided by" 表示该配置参数定义的作用域: `{file:/home/hp/checkout/hello/}default-aea33a/test:fullClasspath ` (fullClasspath 配置在作用域任务维度为test，作用域项目维度为{file:/home/hp/checkout/hello/}default-aea33a的作用域中)

"Dependencies": [配置参数]()章节解释

"Delegates": 表示如果某个key没有定义，将按照以下路劲搜索：

* 两个配置作用域（`runtime:fullClasspath`, `compile:fullClasspath`），在这些作用域中的key，项目维度没有指定默认是当前项目，任务维度没有指定默认是任务全局作用域
* 配置维度为全局的作用域（`*:fullClasspath`），项目维度没有指定默认是当前项目，任务维度没有指定默认是任务全局作用域
* 项目维度设置`{.}`或者ThisBuild (表示工程级别的作用域，没有指定项目)
* 项目维度设置为全局作用域(`*/test:fullClasspath`)(注意：当没有指定项目的时候表示当前项目，这块代表的意思是全局作用域，比如 `*/test:fullClasspath` 和 `test:fullClasspath`代表的意义不一样)
* 项目和配置维度都为全局的作用域 (`*/*:fullClasspath`), 任务维度没有指定，所以当设定为` */*:fullClasspath` 作用域时，在作用域的三个维度上都为全局的

运行 `inspect fullClasspath`（对比上一个例子`inspect test:fullClasspath`） 会发现返回的结果有所不同，这是因为当不指定配置维度的作用域时，sbt将`inspect fullClasspath`自动探测为 `compile.inspect compile:fullClasspath`执行.


### 如何在工程构建中定义作用域

如果单独在build.sbt中创建一个key，这个key作用域的项目维度将为当前项目，配置和任务维度将为全局作用域:

```
name := "hello"
```

运行命令`inspect name`将看到"Provided by"为：`{file:/home/hp/checkout/hello/}default-aea33a/*:name`， 表示作用域项目维度为 `{file:/home/hp/checkout/hello/}default-aea33a`, 作用域任务维度为`*`(全局) ， 作用域任务维度没有指定默认代表全局,
build.sbt 定义是针对单个项目的，所以“当前项目”指的就是当前build.sbt定义的项目(对于[多项目构建](multi_project.html),每个项目有对应一个build.sbt)

所有的key都有一个方法用来设定作用域，其参数可以为作用域的任何一个维度的对象实例。例如，可以通过如下方式将`name`配置设置为配置维度为`Compile` 的作用域：

```
name in Compile := "hello"
```

或者可以将`name`配置设置为任务维度为`packageBin`的作用域（当然这例子有点不合适）

```
name in packageBin := "hello"
```

当然了也可以为一个key指定多个作用域维度,例如将key `name` 同时指定配置维度和任务维度：

```
name in (Compile, packageBin) := "hello"
```

也可以指定一个key的作用域为全局：

```
name in Global := "hello"
```

(`name in Global` 隐式将作用域转化为一个对于所有维度都为全局的作用域，默认情况下任务和配置维度的作用域已经是全局的了，但是这块会影响项目维度的作用，因为其隐式转化为`*/*:name`而不是`{file:/home/hp/checkout/hello/}default-aea33a/*:name`)

如果没有用过Scala语言，理解 `in` 和 `:=` 这两个方法很重要，推荐用scala语法形式配置，但是也可以用Java语法形式配置：

```
name.in(Compile).:=("hello")
```

### 什么时候指定作用域

当定义一个key在默认作用域下会有问题时需要指定作用域，例如`compile`任务类型的配置，默认作用域是在 `Compile`或`Test`配置维度下，不会存在于其他作用域中。

如果修改`compile`这个任务配置的值需要指定其作用域，修改语句如`compile in Compile`或`compile in Test`, 如果单独写作为`compile` sbt 会重新创建一个作用域为当前项目的任务配置，而不是去修改在配置维度作用域下的标准compile任务配置。

如果得到一个错误信息“Reference to undefined setting”， 一般情况下是因为配置项指定作用域失败或者指定了一个错误的作用域。当定义的key可能在其他作用域中已经定义会接收到“Did you mean compile:compile?”错误提示信息

一般会简单的认为配置项就是一个key-value键值对，其实对于所有的配置项都会包含一个key-value 和一个对应的作用域（作用域有三个维度），如配置表达式：` packageOptions in (Compile, packageBin)`, 当配置为`packageOptions`也是一个合法的配置项，只是该配置项的作用域将是默认的作用域(项目维度为当前项目，任务和配置维度为全局)

