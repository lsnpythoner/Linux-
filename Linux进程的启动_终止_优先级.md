---
title: Linux进程的启动_终止_优先级
tags: Linux,
grammar_cjkRuby: true
---
[TOC]
Linux的进程启动的方式不外乎两种方式:调度启动和手动启动,调度启动就是我们经常使用的任务计划,而手动启动则是由用户输入命令,然后Linux执行的一个过程,分为前台启动和后台启动.
### 前台启动
手动启动一个进程,比如输入一个`ls`,`pwd`等命令.它的特点就是会一直占据着终端的窗口,直至完成.一般适合运行时间较短,需要与用户进行交互的程序.
例如,前台启动会占据终端窗口,直至命令运行完成.
```bash
[root@Centos ~]# ll
total 20
drwxr-xr-x. 4 root root 4096 Nov 26  2016 configure
drwxr-xr-x. 3 root root 4096 May 17 14:38 interface
drwxr-xr-x. 2 root root 4096 Nov 27  2016 Script
drwxr-xr-x. 4 root root 4096 Nov 26  2016 speed
drwxr-xr-x. 2 root root 4096 May 17 14:37 test
[root@Centos ~]# 
```
### 后台启动
与前台启动对应,不管进行是否完成,都会立即返回到shell提示符下,在窗口页面等待他完成.好处是,你可以在它后台运行的时候,继续运行其他命令.如果一个进行比较耗时,也不需要用户的交互,可以考虑使用后台启动.要以后台的方式启动一个进程,只要在运行的命令后面添加`&`即可,例如:
```bash
[root@Centos hw]# ./hw_Inter_Stauts.sh &
[1] 12158
[root@Centos hw]# ps -ef |grep '12158'
root      12787  12030  0 10:25 pts/0    00:00:00 grep 12158
[1]+  Done                    ./hw_Inter_Stauts.sh
[root@Centos hw]# 
```
`终端只会告知进程id,可以通过id查看进程的运行结果`
使用`jobs`命令,可以看到系统当前正在运行的所有后台进程:
```bash
[root@Centos hw]# ./hw_Inter_Stauts.sh &
[1] 12792
[root@Centos hw]# jobs
[1]+  Done                    ./hw_Inter_Stauts.sh
[root@Centos hw]# 
```
需要特别注意的是,如果用户退出终端,该用户执行的所有程序全部会结束,包括正在执行的后台程序.
可以使用`nohup`命令,保证命令运行的后台进程不会因此结束:
```bash
[root@Centos hw]# nohup ping www.baidu.com &
[1] 13426
[root@Centos hw]# nohup: ignoring input and appending output to `nohup.out'

[root@Centos hw]# jobs
[1]+  Running                 nohup ping www.baidu.com &
[root@Centos hw]# 
```
`可以使用fg %n关闭nohup命令`
### 终止进程
#### 前台进程-直接Ctrl+C就可以了
如果是后台进行,需要使用`kill`来终止进程,需要是用`ps`命令来去获取进行id,然后用`kill`命令杀掉进程,常用选项`-s`signal顾名思义,信号名或者信号代码,查看所有的信号代码.
```bash
[root@Centos ~]# kill -l
 1) SIGHUP	 2) SIGINT	 3) SIGQUIT	 4) SIGILL	 5) SIGTRAP
 6) SIGABRT	 7) SIGBUS	 8) SIGFPE	 9) SIGKILL	10) SIGUSR1
11) SIGSEGV	12) SIGUSR2	13) SIGPIPE	14) SIGALRM	15) SIGTERM
16) SIGSTKFLT	17) SIGCHLD	18) SIGCONT	19) SIGSTOP	20) SIGTSTP
21) SIGTTIN	22) SIGTTOU	23) SIGURG	24) SIGXCPU	25) SIGXFSZ
26) SIGVTALRM	27) SIGPROF	28) SIGWINCH	29) SIGIO	30) SIGPWR
31) SIGSYS	34) SIGRTMIN	35) SIGRTMIN+1	36) SIGRTMIN+2	37) SIGRTMIN+3
38) SIGRTMIN+4	39) SIGRTMIN+5	40) SIGRTMIN+6	41) SIGRTMIN+7	42) SIGRTMIN+8
43) SIGRTMIN+9	44) SIGRTMIN+10	45) SIGRTMIN+11	46) SIGRTMIN+12	47) SIGRTMIN+13
48) SIGRTMIN+14	49) SIGRTMIN+15	50) SIGRTMAX-14	51) SIGRTMAX-13	52) SIGRTMAX-12
53) SIGRTMAX-11	54) SIGRTMAX-10	55) SIGRTMAX-9	56) SIGRTMAX-8	57) SIGRTMAX-7
58) SIGRTMAX-6	59) SIGRTMAX-5	60) SIGRTMAX-4	61) SIGRTMAX-3	62) SIGRTMAX-2
63) SIGRTMAX-1	64) SIGRTMAX	
[root@Centos ~]# 
```
`代码很多,通过都是9或者15,这两个表示终止进程运行`
比如要终止我们后台的进程,可以如下:
```bash
[root@Centos ~]# ping www.baidu.com > /tmp/ping &
[1] 13550
[root@Centos ~]# ps -ef | grep '13550'
root      13550  13436  0 10:42 pts/0    00:00:00 ping www.baidu.com
root      13552  13436  0 10:42 pts/0    00:00:00 grep 13550
[root@Centos ~]# 
[root@Centos ~]# kill -9 13550
[root@Centos ~]# ps -ef | grep '13550'
root      13556  13436  0 10:42 pts/0    00:00:00 grep 13550
[1]+  Killed                  ping www.baidu.com > /tmp/ping
[root@Centos ~]#
```
`-9表示发送杀死进程的信号`
**如果使用这个命令都无法终止,那么可能这些进程处于僵死状态,需要通过重启计算机解决**
#### killall
如果我们需要杀死一个程序运行的所有进程,使用`kill`命令就比较麻烦了,因为一个程序通常会包含很多的进程,因此,我们可以使用`killall`命令来杀死程序的所有进程
命令格式:`killall`进程名
```bash
[root@Centos ~]# ping www.baiud.com >/tmp/ping &
[1] 13628
[root@Centos ~]# ping www.163.com >/tmp/ping163 &
[2] 13629
[root@Centos ~]# ps -l
F S   UID    PID   PPID  C PRI  NI ADDR SZ WCHAN  TTY          TIME CMD
4 S     0  13592  13588  0  80   0 - 27119 wait   pts/3    00:00:00 bash
4 S     0  13628  13592  0  80   0 - 27394 skb_re pts/3    00:00:00 ping
4 S     0  13629  13592  0  80   0 - 27394 poll_s pts/3    00:00:00 ping
4 R     0  13630  13592  0  80   0 - 27035 -      pts/3    00:00:00 ps
[root@Centos ~]# 
```
`杀死所有ping命令的进程`
```bash
[root@Centos ~]# ps -l
F S   UID    PID   PPID  C PRI  NI ADDR SZ WCHAN  TTY          TIME CMD
4 S     0  13592  13588  0  80   0 - 27119 wait   pts/3    00:00:00 bash
4 S     0  13628  13592  0  80   0 - 27394 poll_s pts/3    00:00:00 ping
4 S     0  13629  13592  0  80   0 - 27394 poll_s pts/3    00:00:00 ping
4 R     0  13631  13592  0  80   0 - 27035 -      pts/3    00:00:00 ps
[root@Centos ~]# killall ping
[1]-  Terminated              ping www.baiud.com > /tmp/ping
[2]+  Terminated              ping www.163.com > /tmp/ping163
[root@Centos ~]# ps -l
F S   UID    PID   PPID  C PRI  NI ADDR SZ WCHAN  TTY          TIME CMD
4 S     0  13592  13588  0  80   0 - 27119 wait   pts/3    00:00:00 bash
4 R     0  13635  13592  0  80   0 - 27033 -      pts/3    00:00:00 ps
[root@Centos ~]# 
```
这是针对进程来进程操作,如果一个文件被很多人杀掉,方便我们队这个文件进行操作和处理,可以使用`fuser`命令,命令格式如下:`fuser -k filename`
```bash
[root@Centos ~]# fuser -m /tmp/ping
/tmp/ping:               1rce     2rc     3rc     4rc     5rc     6rc     7rc     8rc     9rc    10rc    11rc    12rc    13rc    14rc    15rc    16rc    17rc    18rc    19rc    20rc    21rc    22rc    23rc    24rc    25rc    26rc    27rc    28rc    29rc    30rc    31rc    32rc    33rc    38rc    39rc    41rc    42rc    73rc   152rc   153rc   158rc   159rc   160rc   288rc   290rc   307rc   308rc   320rc   400rce   778rc   889rc   932rc   933rc   934rc   935rc  1030rc  1287rce  1312rce  1354rce  1369rce  1380rce  1386rce  1402rce  1404rce  1408rce  1434rce  1459rce  1468rce  1469rce  1510rce  1514rce  1535rce  1560rce  1568rce  1672rce  1684rce  1701rce  1709rce  1720rce  1733rce  1740rce  1745rce  1747rce  1749rce  1751rce  1757rce  1761rce  1776rce  1779rce  1795rce  1865rce  1871rce  1911rce  1922e  1928rce  2312rce  2322rce  2330rce  2331rce  2401rce  2409rce  2411rce  2414rce  2425rce  2433rce  2438rce  2441rce  2442rce  2444rce  2458rce  2459rce  2461rce  2462rce  2465rce  2466rce  2467rce  2471rce  2472rce  2474rce  2476rce  2483rce  2484rce  2486rce  2491rce  2498rce  2507rce  2517rce  2528rce  2537rce  2573rce  2584rce  2622rce  2627rce  2629rce  2632rce  2676rce  2683rce  2695rce  2705rce  2710rce  2711rce  2712rce 13422rce 13473rce 13477rce 13588rce 13592rce
[root@Centos ~]# 
```
### 优先级
在Linux系统中,每个进程在执行时都会有一个优先级,等级越高,进程获取CPU的时间越长,处理就会更快.进程的优先级范围是`-20-19`其中 **-20** 最高,**19** 最低,默认级别为0,使用`nice`和`renice`命令可以更改优先级
```bash
[root@Centos ~]# nice -19 vim fiel01 &
[1] 13700
[root@Centos ~]# nice --19 vim file01 &
[2] 13701

[1]+  Stopped                 nice -19 vim fiel01
[root@Centos ~]# ps -l
F S   UID    PID   PPID  C PRI  NI ADDR SZ WCHAN  TTY          TIME CMD
4 S     0  13592  13588  0  80   0 - 27119 wait   pts/3    00:00:00 bash
0 T     0  13700  13592  0  99  19 - 35363 signal pts/3    00:00:00 vim
4 T     0  13701  13592  0  61 -19 - 35363 signal pts/3    00:00:00 vim
4 R     0  13705  13592  0  80   0 - 27035 -      pts/3    00:00:00 ps

[2]+  Stopped                 nice --19 vim file01
[root@Centos ~]# nice --19 vim file01 &
[3] 13706
[root@Centos ~]# ps -l
F S   UID    PID   PPID  C PRI  NI ADDR SZ WCHAN  TTY          TIME CMD
4 S     0  13592  13588  0  80   0 - 27119 wait   pts/3    00:00:00 bash
0 T     0  13700  13592  0  99  19 - 35363 signal pts/3    00:00:00 vim
4 T     0  13701  13592  0  61 -19 - 35363 signal pts/3    00:00:00 vim
4 T     0  13706  13592  0  61 -19 - 35363 signal pts/3    00:00:00 vim
4 R     0  13709  13592  2  80   0 - 27035 -      pts/3    00:00:00 ps

[3]+  Stopped                 nice --19 vim file01
[root@Centos ~]# ps -l
F S   UID    PID   PPID  C PRI  NI ADDR SZ WCHAN  TTY          TIME CMD
4 S     0  13592  13588  0  80   0 - 27119 wait   pts/3    00:00:00 bash
0 T     0  13700  13592  0  99  19 - 35363 signal pts/3    00:00:00 vim
4 T     0  13701  13592  0  61 -19 - 35363 signal pts/3    00:00:00 vim
4 T     0  13706  13592  0  61 -19 - 35363 signal pts/3    00:00:00 vim
4 R     0  13710  13592  0  80   0 - 27034 -      pts/3    00:00:00 ps
[root@Centos ~]# 
```