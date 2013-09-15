##反编译apk文件遇到的问题

##困扰了一天啊，终于搞定了

以前有反编译过安卓的apk文件，没怎么深究，现在要用到，遇到了一个问题，困扰了一整天才解决了，真是不简单啊！

反编译修改smali文件后，怎么都不能重新编译成apk文件了，老提示
brutexception：could not exec command：[aapt,p,--min-sdk-version,9......]
等等一堆错误提示，google查找，很多都说是aapt的版本太低，用android sdk里的aapt替换掉工具里的appt，我试了半天也不行，重新下了最新的apktool也不行，试了无数遍都不行，真的要崩溃了，后来查到貌似和framework-res.apk的版本有关系，可能是app太新，老的framework不兼容了，最后又是千辛万苦找到了一个4.0的framework，用
 `apktool if framework path` 导入，终于可以了，开心。整整花了一整天怎么感觉写出来也没什么大不了的，真是一窍难得！！
