##eclipse环境下的andriod开发环境搭建##

前几天在笔记本上搭建了android的开发环境，感觉太慢了，电脑的配置有点赶不上时代了，于是考虑在实验室的8核台机上搭建，台机配置很好，是64位的ubuntu13.04的系统，搭建好sdk和adt后，发现emulator根本启动不了，提示no such a file or dirctory,太奇怪了，路径明明就是对的，开始怀疑是不是sdk安装的不全，在windows上遇到过这种情况，于是打开sdk manager把可用的都安装上了，重启eclipse之后还是不行，
非常郁闷，从早上装到现在，头昏闹胀了，后来终于搜索到了相关的问题解决方案，是在**stackoverflow**上看到的，真是一个大神云集的地方，相比起来，百度知道就是一个灌水的地方，基本得不到什么有用的信息。

原来，由于是我用的是64bit的系统，而Android sdk只有32bit的程序，需要安装ia32-libs，才能使用。

    sudo apt-get install ia32-libs

问题貌似解决了，可是这个软件真是难装，花了差不多一下午，速度奇慢无比，最后还没安装成功，实在没法，还是stackoveflow给力，又见一个大神的回复：

On (K)Ubuntu you need following 32-bit packages:

    sudo apt-get install libstdc++6:i386 libgcc1:i386 zlib1g:i386 libncurses5:i386

for running the emulator you need that additional package:

    sudo apt-get install libsdl1.2debian:i386

一共就十几兆的东西，一会就安装完成了，重启eclipse，哈哈，avd成功启动！！激动啊！

还有一件事：我最开始创建了一个android程序，但是其中的R文件一直没有，导致程序处在出错状态，在查这个错误的时候很幸运也得到了答案。在android avd manager中没有一个avd时，eclipse是不会自动生成R.java文件的......可见错误的出现有很多相通的地方.

####总结####

* 遇到问题一定多问google大神，百度只适合查查生活上的问题。。

* stackoverflow真是一个计算机方面的神站，各路大神云集的地方，有问题就找他。

* 思路一定要开阔，遇到问题不能着急，伤身！！

* 大防火墙太坑爹，本来google的sdk manager用起来应该很方便的，可是在国内速度太慢，时有时无的信号给我们安装增加了难度。

###在8核的台机上运行emulator快如飞，太爽了，相比起来笔记本太不给力了###
