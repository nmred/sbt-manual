# Lib库依赖

添加Lib库依赖关系有两种方式：

* 非管理依赖方式，是通过将依赖的Jar包放到项目的lib目录
* 管理依赖方式，是在工程构建配置中配置依赖关系，sbt会自动从托管代码库中下载依赖库

### 非管理依赖方式


很多人用管理依赖的方式替代非管理方式，其实非管理方式用起来非常方便。非管理依赖方式的工作原理就是将jar包放到lib目录下，sbt会自动的将其添加到classpath中。也可以将一些测试依赖放到lib目录下，如[ScalaCheck](http://scalacheck.org/)、[Specs2](http://specs2.org/)和[ScalaTest](http://www.scalatest.org/)这些依赖包


lib目录下载的依赖在所有的classpath中有效（对于编译、测试、运行或命令终端），如果想修改某个配置维度的作用域的classpath配置，需要按照如下修改方式`dependencyClasspath in Compile` 或 `dependencyClasspath in Runtime`

对于非管理方式的依赖不需要在build.sbt额外配置，但是如果想自定义一个依赖目录而不是默认的lib目录时，可以通过修改`unmanagedBase`参数配置, 比如用目录`custom_lib`替换`lib`目录

```
unmanagedBase := baseDirectory.value / "custom_lib"
```

其中`baseDirectory`是项目的根目录，修改`unmanagedBase`参数配置依赖`baseDirectory`,关于参数配置的依赖可以参考[配置参数的方法](kind_setting.html)

在非管理依赖方式中还提供了一个任务配置`unmanagedJars`用来列举`unmanagedBase`所有的jar包的，在多项目构建中或更加复杂的构建中可能需要修改`unmanagedJars`配置来完成。例如，在Compile这个配置维度作用域下要清空所有非管理方式的jar包，可以利用如下配置：

```
unmanagedJars in Compile := Seq.empty[sbt.Attributed[java.io.File]]

```

### 管理依赖方式

sbt 利用[Apache Ivy](https://ant.apache.org/ivy/)方式管理依赖包，如果熟悉`Ivy`或`Maven`的话，理解sbt的管理依赖包将会非常的容易。

#### `libraryDependencies` 参数配置

一般情况下只需要通过配置`libraryDependencies`参数配置即可设置依赖的包，也可以通过编写`Maven`的POM配置文件或`Ivy`的配置文件来扩展包依赖的功能。

以下是申明一个包依赖关系， 其中`groupId`，`artifactId`和`revision `是字符串类型

```
libraryDependencies += groupID % artifactID % revision
```

或用如下申明，其中`configuration`是一个字符串或者一个配置维度实例
```
libraryDependencies += groupID % artifactID % revision % configuration

```

libraryDependencies这个配置key在Keys中声明语句如下

```
val libraryDependencies = settingKey[Seq[ModuleID]]("Declares managed dependencies.")
```

从上述 libraryDependencies key 申明语句中可以看出其值是接收一个由ModuleID对象构成的序列。在申明依赖关系的语句中有`%`方法，该方法是用字符串创建一个ModuleID类型的对象。

当然sbt必须知道所配置的依赖库到什么地方下载，如果配置的依赖库在默认的远程库中存在将直接下载，比如`Apache Derby`就是在标准远程库`Maven2`中：

```
libraryDependencies += "org.apache.derby" % "derby" % "10.4.1.3"

```

如果配置到build.sbt 并且执行`update`， sbt将自动将其下载到`~/.ivy2/cache/org.apache.derby/`目录下(由于`compile`任务配置会依赖`update`这个任务，所以一般情况不用手动执行update)

当然，也可以用 `++=` 方法一次性添加一个依赖列表:

```
libraryDependencies ++= Seq(
  groupID % artifactID % revision,
  groupID % otherID % otherRevision
)
```

在少数情况下可能也会用到`:=`赋值方法

#### 指定依赖库Scala版本

如果用 `groupID %% artifactID % revision`而不是`groupID % artifactID % revision` 方式配置依赖关系(二者区别在于前者的groupId后的是两个`%`)， sbt将会添加当前scala版本到依赖的报名后，这个只是一个快捷的方式，你也可以直接硬编码scala版本：

```
libraryDependencies += "org.scala-tools" % "scala-stm_2.11.1" % "0.3"

```

假如当前构建项目的`scalaVersion`为2.11.1,如下方式和上述的结果是一样(利用`%%`方式添加依赖库的Scala版本)

```
libraryDependencies += "org.scala-tools" %% "scala-stm" % "0.3"

```

在多个Scala版本下构建项目，可以使用这种方法来匹配二进制兼容的对应依赖包。

复杂的情况是经常有依赖包对于不同的Scala版本间有细微的差别，所以如果依赖包在`2.10.1 `版本下存在，但是当前项目scalaVersion := "2.10.4", 将会发现用`%%`是获取不到依赖包的，这时需要确定该版本的依赖包是否可用，并且可以通过硬编码版本的方式添加依赖

#### Ivy 自动匹配版本

在配置`groupID % artifactID % revision` 中`revision`不是一定要指定一个固定的版本号，Ivy 可以自动选择一个高版本包根据指定的限定条件，例如如果要替换一个固定版本"1.6.1"，可以通过指定"latest.integration","2.9.+"或"[1.0,)"来替换，更多的配置请参看Ivy配置文档

#### 远程依赖库地址管理

不是所有的包在同一个远程库中都存在，sbt默认用的是标准的`Maven2`远程库，如果依赖的包不在默认的远程库中，需要手动指定一个远程库供Ivy查找。

添加远程库的方法如下：

```
resolvers += name at location
```

例如：

```
resolvers += "Sonatype OSS Snapshots" at "https://oss.sonatype.org/content/repositories/snapshots"

```

`resolvers` 这个配置项在 `Keys`中的定义如下：

```
val resolvers = settingKey[Seq[Resolver]]("The user-defined additional resolvers for automatically managed dependencies.")

```

在添加远程库配置语句中的 `at` 方法是将字符串转化为`Resolver`对象。

sbt 也支持搜索本地的`Maven`依赖库，配置如下：

```
resolvers += "Local Maven Repository" at "file://"+Path.userHome.absolutePath+"/.m2/repository"

```

也可以简写为：

```
resolvers += Resolver.mavenLocal
```

#### 重写默认的远程依赖库

参数配置`resolvers`中是不包含默认的远程库配置的，仅仅用来配置添加远程库地址，sbt 最终是通过合并`resolvers`配置和`externalResolvers`配置来得到远程库地址集合，所以如果要修改默认远程库的话需要修改参数配置`externalResolvers`

#### 配置维度的作用域依赖库

经常在测试代码中会用到依赖库（在src/test/scala目录中的代码将通过配置维度为Test作用域的配置来编译）但在项目代码中不会用到。

如果只想在编译测试代码的时候加入到classpath而在编译项目代码的时候不加入，可以通过添加% "test"指定配置维度的作用域来限定：

```
libraryDependencies += "org.apache.derby" % "derby" % "10.4.1.3" % "test"

```

也可以指定配置维度的一个实例对象来限定：

```
libraryDependencies += "org.apache.derby" % "derby" % "10.4.1.3" % Test

```

那么，当限定到某个配置维度作用域时利用命令`show compile:dependencyClasspath`查看配置时将不会看到` derby `jar包，但是如果查看`show test:dependencyClasspath`将会看到`derby` jar包在列表中。

常见的，测试相关的包[ScalaCheck](http://scalacheck.org/)、[Specs2](http://specs2.org/)和[ScalaTest](http://www.scalatest.org/)将指定% "test"来限定在Test配置维度下使用。


