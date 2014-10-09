## 运行 SBT

### 交互模式

在根目录中运行 `sbt` 命令不带任何参数将进入交互模式

```
$ sbt
```

交互模式有一个命令输入功能（可以用 Tab 补全和历史命令）， 例如，当输入 `compile` 时：

```
> compile
```

如果再次编译只需按 向上键 + 回车键
如果运行该项目输入 `run`
如果退出交互模式输入 `exit` 或用快捷键 `Ctrl+D`(Unix) 或 `Ctrl+Z`(Windows)

### 批量脚本模式

你也可以在批量脚本模式下运行 sbt, 指定一个用空格分割的一系列命令作为参数. 对于 sbt 命令本身也可以指定对应的参数，将命令和命令参数用双引号括起来，其中一个参数为命令，其余的为命令参数，例如：

```
$ sbt clean compile "testOnly TestA TestB"
```

在这个例子中，testOnly有两个参数分别是 TestA 和 TestB. 这个命令将按照 `clean, compile, testOnly` 顺序来执行.

### 持续构建于测试

为了提供 编辑-编译-测试 整个周期的效率，可以使用 sbt 的自动触发编译和运行过程当保存源代码文件的时候。使一个或多个源代码文件修改后可以自动指定对应的命令，只需在对应命令前加 `~` 前缀. 例如，在交互模式下：

```
> ~ compile
```

按回车键定制监视文件的改变

### 常用命令

`clean`

删除所有构建生成的文件(在target目录中)

`compile`

编译项目源代码(编译 src/main/scala 和 src/main/java 目录下的源代码)

`test`

编译并运行所有的测试用例

`console`

启动一个 Scala 语言交互模式， sbt 在启动的时候会指定依赖的所有classpath, 返回 sbt 可以用 `:quit` 、 `Ctrl+D`(Unix) 和 `Ctrl+Z`(Windows)

`run <arguments>*`

运行项目在虚拟机中

`package`

创建一个 jar 包其中包含 `src/main/resources ` 和编译 `src/main/scala` 或 `src/main/java` 目录的 `class` 文件

`help <command>`

显示指定命令的帮助信息，如果没有指定命令将显示所有的命名的摘要信息。

`reload`

重新加载配置文件(build.sbt， project/.scala 和 project/.sbt 文件)， 当修改配置文件的时候需要执行

### Tab 补全

在交互模式下 sbt 支持 Tab 补全功能，当按一次 Tab 键是 sbt 会显示所有可能匹配的子集命令，当按多次 Tab 后将显示多个可能匹配的命令进行选择的提示，和 Unix tab 补全规则基本一致

### 历史命令

在交互模式下可以 sbt 会记录历史命令，甚至是退出sbt 后重启历史命令还会存在， 利用历史命令简单的方法是按 "向上键 + 回车键" 调用上一次执行的命令，以下是所有执行的历史命令调用方法：

`!`

显示历史命令的帮助信息

`!!`

再次执行上一个命令

`!:`

显示所有的历史命令

`!:n`

显示最后的 n 个历史命令

`!n`

执行index 为 n的命令，index为执行 `!:`命令显示的index

`!-n`

执行第 n 个命令的前一个命令

`!string`

执行以 'string' 开头的最近的命令

`!?string`

执行包含 'string' 字符串的命令

