### 手动安装 SBT

#### Unix

将[sbt-launch.jar][sbt-jar]包放到目录 ~/bin中

创建一个运行jar包的脚本 ~/bin/sbt, 脚本内容为：

```
SBT_OPTS="-Xms512M -Xmx1536M -Xss1M -XX:+CMSClassUnloadingEnabled -XX:MaxPermSize=256M"
java $SBT_OPTS -jar `dirname $0`/sbt-launch.jar "$@"
```

确保脚本有执行权限

```
$ chmod u+x ~/bin/sbt
```

## Windows

[sbt-jar]:(http://repo.typesafe.com/typesafe/ivy-releases/org.scala-sbt/sbt-launch/0.13.5/sbt-launch.jar)
