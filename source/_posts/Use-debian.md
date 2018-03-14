---
title: Use-debian
date: 2015-03-06 9:20
tags:
 - Linux
---

## 1.debian的安装：

下载官网的Debian DVD版本用Win32DiskImager软件做成U盘启动安装(或使用 GNOME Multi-Writer)，
开机设置BIOS的USB启动直接进入安装界面，还可以选择图形界面安装很方便.

## 2.安装网卡驱动实现联网

安装过程中会提示你的有限和无线网卡驱动未安装信息。我的是这两个：rtl_nic/rtl8168d-2.fw和rtlwifi/rtl8192sefw.bin谷歌搜索 可得到链接debian的deb包为firmware-realtek_0.43_all.deb，很方便。安装后重启就能链接无线上网了。

## 3：配置源:

```bash
sudo nano /etc/apt/sources.list
deb http://ftp.cn.debian.org/debian/ testing main non-free contrib
deb-src http://ftp.cn.debian.org/debian/ testing main non-free contrib
deb http://ftp.cn.debian.org/debian/ testing-proposed-updates main non-free contrib
deb-src http://ftp.cn.debian.org/debian/ testing-proposed-updates main non-free contrib
deb http://ftp.cn.debian.org/debian/ testing-updates main non-free contrib
deb-src http://ftp.cn.debian.org/debian/ testing-updates main non-free contrib
deb http://security.debian.org/ testing/updates main contrib non-free

使用Sid，则可将更新列表改为：
deb http://ftp.cn.debian.org/debian/ sid main contrib non-free
deb http://ftp.cn.debian.org/debian/ sid main contrib non-free
```

## 4.debian更新:

```bash
$ sudo aptitude update
$ sudo aptitude safe-upgrade
$ sudo aptitude full-upgrade
```

## 5.驱动问题

```bash
sudo aptitude install firmware-linux-nonfree
```

## 6.安装sudo 使普通用户有系统管理的权利

```bash
root# aptitude install sudo
root# chmod +w /etc/sudoers          授予sudoers 写的权限，否则为只读。
root# nano /etc/sudoers               编辑  /etc/sudoers
找到root 行并在下行输入你的账户,在账户后面把root后面的复制下来就行了,然后保存.
root# chmod 0440 /etc/sudoers        去掉sudoers 写的权限改为只读。
root# exit                           退到普通账户。
```
#### sudo找不到命令：修改sudo的PATH路径

sudo有时候会出现找不到命令，而明明PATH路径下包含该命令，让人疑惑。其实出现这种情况的原因，主要是因为当 sudo以管理权限执行命令的时候，linux将PATH环境变量进行了重置，当然这主要是因为系统安全的考虑，但却使得sudo搜索的路径不是我们想要的PATH变量的路径，当然就找不到我们想要的命令了。两种方法解决该问题：

首先，都要打开sudo的配置文件：sudo visudo

- 1.可以使用 secure_path 指令修改 sudoers 中默认的 PATH为你想要的路径。这个指令指定当用户执行 sudo 命令时在什么地方寻找二进制代码和命令。这个选项的目的显然是要限制用户运行 sudo 命令的范围，这是一种好做法。

- 2.将Defaults env_reset改成 Defaults !env_reset取消掉对PATH变量的重置，然后在.bashrc中最后添加
```bash
alias sudo='sudo env PATH=$PATH'
```
这样sudo执行命令时所搜寻的路径就是系统的PATH变量中的路径，如想添加其他变量也是类似。


## 7.输入法

Fcitx输入法的前端是需要UI动态库支持，您会发现在Fcitx的安装目录中并没有该文件，
这应该是官方打包的时候就已经遗漏的问题，解决方法就是安装UI动态库支持必须的文件，然后重启Debian系统您的问题将会迎刃而解。

```bash
Terminal 输入：apt-get install fcitx-ui-classic && apt-get install fcitx-ui-light
```

## 8.自定义Terminal快捷键

打开系统设置---键盘---快捷键--自定义快捷键
点加号添加一个自定义快捷键   名称根据自己喜欢起名:Terminal  命令:gnome-terminal   然后自定义一个快捷方式就可以了。

## 9.创建快捷方式

默认情况下，自动安装的软件快捷方式保存在/usr/share/applications目录下，要创建桌面快捷方式，只需要右键-复制-桌面。
上面的方法是通过系统自动安装软件后实现的，有时候我们自己会从网上下载一些软件手动安装，该怎样创建软件的桌面快捷方式？
这里以Eclipse 为例，首先到官网下载Eclipse软件包，直接解压在某个目录下，双击其中的eclipse文件，就可以启动eclipse了，
不过如果每次要打开eclipse，都要从安装目录启动，是不是有些麻烦？依照下面的操作，
在/usr/share/applications目录下创建一个文件名修为eclipse.desktop ，并添加执行权限。

```bash
模板：

[Desktop Entry]
Encoding=UTF-8
Categories=Development;
Comment[zh_CN]=
Comment=
Exec=/home/xukun/eclipse/eclipse
GenericName[zh_CN]=IDE
GenericName=IDE
Icon=/home/xukun/eclipse/icon.xpm
MimeType=
Name[zh_CN]=eclipse
Name=eclipse
Path=
StartupNotify=true
Terminal=false
Type=Application
X-DBUS-ServiceName=
X-DBUS-StartupType=
X-KDE-SubstituteUID=false
X-KDE-Username=xukun

实例

[Desktop Entry]
Encoding=UTF-8
Name=eclipse
Comment= Eclipse for c++
Exec=/home/xukun/eclipse/eclipse
Icon=/home/xukun/eclipse/icon.xpm
Terminal=false
Type=Application
Categories=Application;Development;
```

关注3个地方，分别为Exec=软件执行文件的路径，Icon=快捷方式图标（如果有的话），Name=快捷方式名称。
根据自己软件按转的位置修改代码，以上代码保存后，发现 左上角的 活动 应用程序里面已经有了，
想添加到左侧的快速启动栏，就右键图标，添加到收藏夹就行了，要创建桌面快捷方式，只需要右键-复制-桌面 就Ok。

## 10.Lantern

直接安装下载好的安装包。下载地址：https://github.com/getlantern/lantern
若安装了Lantern之后，不能启动起来，一闪而过，命令行里启动，包如下错误：

/.lantern/bin/lantern: error while loading shared libraries: libappindicator3.so.1: cannot open shared
  object file: No such file or directory

尝试：
```bash
$ apt-cache search libappindicator3
gir1.2-appindicator3-0.1 - Typelib files for libappindicator3-1
libappindicator3-1 - allow applications to export a menu into the panel -- GTK3 version
libappindicator3-dev - allow applications to export a menu into the panel -- GTK3 development

安装  apt-get install libappindicator3-1
```

## 11.uget+aria2

```bash
 aptitude install uget
 aptitude install aria2
 ```
然后启用aria2插件

## 12.开发软件

**Git, rust，VSCode，GNU工具链**

- GNU make：用于编译和构建的自动工具； 检验：make --version/make -v

- GNU编译器集合（GCC）：一组多种编程语言的编译器 检验：gcc --version/gcc -v

- G++：C++语言的编译器 检验：g++ --version/g++ -v

- GNU Binutils：包含链接器、汇编器和其它工具工具集

- GNU Debugger（GDB）：代码调试工具； 检验：gdb --version/gdb -v

**GNU构建系统（autotools）:**

- Autoconf
- Autoheader
- Automake
- Libtool
