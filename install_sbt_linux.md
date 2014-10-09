### Linux 平台安装 SBT

#### 通过通用的包安装

下载 [ZIP][down-zip] 包或 [TGZ][down-tgz] 包解压

#### RPM 和 DEB

* [RPM][down-rpm]包
* [DEB][down-deb]包


> 注意： 请将任何和这两个包相关的问题反馈到[sbt-launcher-package][sbt-launcher]项目 issue

**Gentoo**

In the official tree there is no ebuild for sbt. But there are ebuilds to merge sbt from binaries. To merge sbt from this ebuilds you can do:

```
$ mkdir -p /usr/local/portage && cd /usr/local/portage
$ git clone git://github.com/whiter4bbit/overlays.git
$ echo "PORTDIR_OVERLAY=$PORTDIR_OVERLAY /usr/local/portage/overlays" >> /etc/make.conf
$ emerge sbt-bin
```

> 注意: 有任何和 ebuild 相关的问题请反馈到 [ebuild issue][ebuild]


**手动安装**

参考[手动安装 SBT](install_sbt_manual.html)

[down-zip]:(https://dl.bintray.com/sbt/native-packages/sbt/0.13.6/sbt-0.13.6.zip)
[down-tgz]:(https://dl.bintray.com/sbt/native-packages/sbt/0.13.6/sbt-0.13.6.tgz)
[ebuild]:(https://github.com/whiter4bbit/overlays/issues)
[sbt-launcher]:(https://github.com/sbt/sbt-launcher-package)
[down-rpm]:(https://dl.bintray.com/sbt/rpm/sbt-0.13.6.rpm)
[down-deb]:(https://dl.bintray.com/sbt/debian/sbt-0.13.6.deb)
