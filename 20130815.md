#今天的笔记#
1.tty和pty/pts(pty的实现形式）：tty就是ctrl+alt+f1-fn的虚拟终端代号，tty7就是gen  代号，tty就是代表当前的终端号。pts就是在xwindow下打开的模拟终端，每打开一个ter  minal，就会多出现一个pts(n)。如果有其他的用户处于登录状态，可以踢出该用户，直   接结束掉他的tty就行了：先查看他登录的tty号，用w命令，然后sudo pkill -KILL -t    tty(n)。 
