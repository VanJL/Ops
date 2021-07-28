# sudo

> sudo是系统中一个增强安全的工具，也是一个比较常用到的安全工具，通过sudo我们可以直接使用普通用户来执行一些root用户才能执行的命令等。

## Author

```
Name:Shinefire
Blog:https://github.com/shine-fire/Ops_Notes
E-mail:shine_fire@outlook.com
```



## 原理介绍

以下为sudo命令的执行时的工作原理图，仅供参考

![](sudo.assets/wKioL1M6bkbCEMkwAAG4AtoubNI249.jpg)

​         sudo：
作用：root用户给其他的用户设置好权限，然后让被设置好的用户可以使用一些root用户才能使用的命令，当然是要设置权限的时候设置的那个范围的才行。

不建议使用vim /etc/sudoers
而是建议使用visudo

\-----------------------------------------------------------------------
由于有些软件是会默认调用vi编辑器的，但是vi编辑器是不会有颜色高亮那些的，这个问题的解决方法：
再/etc/profile文件里面加入下面的两行即可：
EDITOR=vim

export EDITOR
\----------------------------
sudo的语法：
USER  MACHINE=COMMANDS
哪个用户在哪些机器上能执行哪些命令
user ALL=NOPASSWD:  这样的话这个用户使用sudo就不需要输入密码了
多个命令之间用,隔开
\--------------------------------------------------------
给组加sudo的话，语法则是：
%groupname  MACHINE=COMMANDS
\--------------------------------------------------------
在vim编辑器的末行模式中输入以下命令可以直接查看该命令的绝对路径:
:!which  ls
\--------------------------------------------------------------
ceshi   ALL=ALL,!/usr/bin/passwd
表示除了passwd命令以外，其他的所有命令都可以执行

==================================

定义别名：（其实就和普通的定义别名alias差不多的）
有别名的话可以自己把一些相关的命令放在一起，然后调用的时候可以直接调用别名就可以了，特别是要给几个用户都要给到相同的几个命令的时候，可以直接写别名就可以了
/etc/sudoers 里面也有一些系统默认就定义好了的别名，不过都被注释了的，要是之后还会不懂就自己去看配置文件就行了

Cmnd_Alias  BIEMING= /usr/bin/command
Cmnd_Alias  是定义别名的方式，然后后面是 别名= 你要选择的命令，多个命令之间用, 隔开即可

=====================================

sudo日志：
/var/log/secure  

=====================================

rhel6中  wheel组是默认注释的，而且拥有和root一样的权限
rhel7中  wheel中是默认打开的，所以最好自己再去注释掉也可以



## 参考文献
