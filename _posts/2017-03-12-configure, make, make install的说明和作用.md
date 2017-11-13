---
layout: post
title: ./configure, make, make install的说明和作用
categories: Linux
description: ./configure, make, make install的说明和作用
keywords: Linux, make, make install, configure
---

Linux下源码包安装命令的作用详解. 以及报错后清除编译文件的命令. 

## 编译安装的3个命令

**./configure**
是用来检测你的安装平台的目标特征的。比如它会检测你是不是有CC或GCC，并不是需要CC或GCC，它是个shell脚本。

**make**
是用来编译的，它从Makefile中读取指令，然后编译。

**make install**
是用来安装的，它也从Makefile中读取指令，安装到指定的位置。

---


1. **configure**: 这一步一般用来生成 Makefile，为下一步的编译做准备，你可以通过在 configure 后加上参数来对安装进行控制，比如代码:./configure –prefix=/usr 意思是将该软件安装在 /usr 下面，执行文件就会安装在 /usr/bin （而不是默认的 /usr/local/bin),资源文件就会安装在 /usr/share（而不是默认的/usr/local/share）。同时一些软件的配置文件你可以通过指定 –sys-config= 参数进行设定。有一些软件还可以加上 –with、–enable、–without、–disable 等等参数对编译加以控制，你可以通过允许 ./configure –help 察看详细的说明帮助。

2. **make**: 这一步就是编译，大多数的源代码包都经过这一步进行编译（当然有些perl或Python编写的软件需要调用perl或python来进行编译）。如果 在 make 过程中出现 error ，你就要记下错误代码（注意不仅仅是最后一行），然后你可以向开发者提交 bugreport（一般在 INSTALL 里有提交地址），或者你的系统少了一些依赖库等，这些需要自己仔细研究错误代码。

3. **make insatll**: 这条命令来进行安装（当然有些软件需要先运行 make check 或 make test 来进行一些测试），这一步一般需要你有 root 权限（因为要向系统写入文件）。

---


## Makefile有关的命令

利用configure所产生的Makefile文件有几个预设的目标可供使用，其中几个重要的简述如下：

**make all**：产生我们设定的目标，即此范例中的可执行文件。只打make也可以，此时会开始编译原始码，然后连结，并且产生可执行文件。

**make clean**：清除编译产生的可执行文件及目标文件(object file，*.o)。

**make distclean**：除了清除可执行文件和目标文件外，把configure所产生的Makefile也清除掉。

**make install**：将程序安装至系统中。如果原始码编译无误，且执行结果正确，便可以把程序安装至系统预设的可执行文件存放路径。如果用bin_PROGRAMS宏的话，程序会被安装至/usr/local/bin这个目录。

**make dist**：将程序和相关的档案包装成一个压缩文件以供发布。执行完在目录下会产生一个以PACKAGE-VERSION.tar.gz为名称的文件。 PACKAGE和VERSION这两个变数是根据configure.in文件中AM_INIT_AUTOMAKE(PACKAGE，VERSION)的定义。在此范例中会产生test-1.0.tar.gz的档案。

**make distcheck**：和make dist类似，但是加入检查包装后的压缩文件是否正常。这个目标除了把程序和相关文件包装成tar.gz文件外，还会自动把这个压缩文件解开，执行 configure，并且进行make all 的动作，确认编译无误后，会显示这个tar.gz文件可供发布了。这个检查非常有用，检查过关的包，基本上可以给任何一个具备GNU开发环境-的人去重新编译。





