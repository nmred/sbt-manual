# 配置参数的方法

### 回顾：配置项

一个工程构建定义一个 `Setting` 类型的列表，通过sbt转化为sbt的描述数据结构（key-value 键值对
），Setting作为一个转化前的输入类型，转化后输出一个 map表。

不同的配置项有不同的转化方法，例如前面的 `:=` 方法。 一个 Setting类型的配置项可以通过 `:=` 转化成一个值为一个常量的map表，例如，转化一个配置项`name := "hello"` 是将 hello 赋值给该配置项的key

Settings must end up in the master list of settings to do any good (all lines in a build.sbt automatically end up in the list, but in a .scala file you can get it wrong by creating a Setting without putting it where sbt will find it).

### 配置值追加操作： += 和 ++=

通过直接赋值方法 `:=` 是最简单的一种转化方法， 所有的配置项还有其他的方法，在 `SettingKey[T]` 中泛型 T 如果是一个序列类型的话，支持追加操作不仅仅是重新赋值替换操作。

* `+=` 操作是追加单个元素到序列中
* `++=` 操作是追加一个序列到序列中

比如, 对于配置 `sourceDirectories in Compile` 值得类型是 `Seq[File]`, 默认情况下这个配置已经有包含了目录`src/main/scala`。如果还想编译目录`source`下的源代码可以添加该目录：

```
sourceDirectories in Compile += new File("source")
```

或者，利用 sbt 包中提供的函数 `file()` 更加方便

```
sourceDirectories in Compile += file("source")
```

file() 函数也是创建 File 对象

也可以用操作符`++=` 一次性添加多个目录：

```
sourceDirectories in Compile ++= Seq(file("sources1"), file("sources2"))
```

其中Seq(a, b, c, ...)是 Scala的标准语法，用来创建一个序列

当然了除了追加操作还支持直接赋值替换配置原值：

```
sourceDirectories in Compile := Seq(file("sources1"), file("sources2"))
```

### 通过其他配置项计算一个配置项

引用其他的任务配置或参数配置的值通过调用任务或参数配置，通过方法`:=`、`+=`或`++=`来引用一个配置

比如，定义一个项目组织和项目名称一样的配置：

```
// organization 接收的值类型和 name一样都是 Setting[String]
organization := name.value
```

或者通过引用项目目录来定义项目名称：

```
// name 是Key[String]类型，baseDirectory是Key[File]
name := baseDirectory.value.getName
```

由于baseDirectory的值类型和name的值类型不一样，需要调用java的java.io.File类库getName 方法转化

sbt同时支持引用多个配置项的值，例如：

```
name := "project " + name.value + " from " + organization.value + " version " + version.value
```

#### 参数配置依赖

在配置`name := baseDirectory.value.getName`中，参数配置 name 依赖配置baseDirectory， 如果在放有上述配置的build.sbt的目录下交互模式运行`inspect name`命令，将返回如下的提示信息：

```
[info] Dependencies:
[info]  *:baseDirectory
```

sbt 是很容易的探测出各个参数配置间的依赖关系的，包括任务配置也可以依赖参数配置，或者任务配置间相互依赖， 例如运行`inspect compile` 会发现 compile 任务配置会依赖参数配置compileInputs， `inspect compileInputs` 还会发现参数配置compileInputs又依赖其他的参数配置，等等。最后会发现形成一个依赖链，这些依赖关系是在sbt编译的时候自动探测计算的。所以所有的依赖关系都是自动计算的，不需要显示的声明。

#### 未定义配置

当用`:=`、`+=`或`++=`方法去引用一个配置，所依赖的配置必须存在，否则sbt会报错误信息"Reference to undefined setting", 如果报错需要确认依赖的配置是否在指定作用域中存在。

也可能会出现循环依赖的配置错误，这个时候sbt会提示错误

#### 任务配置引用参数配置

一个任务配置可以依赖参数配置或其他的任务配置，通过 `def.task`或操作符(`:=`、`+=`或`++=`)引用配置

比如，一个源代码生成任务将依赖项目根目录和编译classpath两个参数配置：

```
sourceGenerators in Compile += Def.task {
  myGenerator(baseDirectory.value, (managedClasspath in Compile).value)
}.taskValue
```

#### 任务配置依赖

如[配置文件 .sbt](build_define.html)所述，一个任务配置用赋值方法`:=`操作的参数是`Setting[Task[T]]`而不是`Setting[T]`等， **任务配置输入参数可以接收一个参数配置作为输入，但是一个参数配置是不可以接收一个任务配置作为输入**

以下为两个配置：

```
val scalacOptions = taskKey[Seq[String]]("Options for the Scala compiler.")
val checksums = settingKey[Seq[String]]("The list of checksums to generate and to verify for dependencies.")
```

（scalacOptions和checksums之间没有任何联系，它们只是配置值得类型相同，但是其中scalacOptions是一个任务配置）

在build.sbt中允许一个任务配置依赖参数配置，比如:

```
// 合法的配置
scalacOptions := checksums.value
```

是合法的配置方法，但是如果将一个参数配置配置成依赖一个任务配置的时候就会报错，因为参数配置在项目加载的时候只计算一次，可以当做常量处理，但是任务配置是在不断的重复的运行的。

```
// 非法的配置
checksums := scalacOptions.value
```

### 追加依赖操作：+= 和 ++=

一些配置可以被追加引用到一个已经存在的配置中，和直接赋值操作`:=`一样.

例如，有一个项目覆盖率报告文件（文件名引用项目名称的参数配置）需要删除，可以通过如下配置：

```
cleanFiles += file("coverage-report-" + name.value + ".txt")
```
