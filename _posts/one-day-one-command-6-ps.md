---
title: 每天一个 Linux 命令（6）： ps
date: 2017-02-13 10:08:57
tags:
 - Linux
category: 
 - Linux
thumbnail: 
author: bsyonline
lede: "没有摘要"
---

ps 是 Process Status 的缩写，ps 命令用来列出 linux 系统中当前运行的进程。

<!-- more -->

Linux 系统中进程有 5 种状态：

1. 运行(正在运行或在运行队列中等待)
2. 中断(休眠中, 受阻, 在等待某个条件的形成或接受到信号)
3. 不可中断(收到信号不唤醒和不可运行, 进程必须等待直到有中断发生)
4. 僵死(进程已终止, 但进程描述符存在, 直到父进程调用 wait4() 系统调用后释放)
5. 停止(进程收到 SIGSTOP, SIGSTP, SIGTIN, SIGTOU 信号后停止运行运行)

要了解某时刻系统进程的运行状态，ps 是最常用的命令之一。

ps 命令格式：
```
Usage:
 ps [options]

Basic options:
  -A, -e               all processes
  -a                   all with tty, except session leaders
   a                   all with tty, including other users
  -d                   all except session leaders
  -N, --deselect       negate selection
   r                   only running processes
   T                   all processes on this terminal
   x                   processes without controlling ttys

Selection by list:
  -C <command>         command name
  -G, --Group <GID>    real group id or name
  -g, --group <group>  session or effective group name
  -p, p, --pid <PID>   process id
         --ppid <PID>  parent process id
  -q, q, --quick-pid <PID>
                       process id (quick mode)
  -s, --sid <session>  session id
  -t, t, --tty <tty>   terminal
  -u, U, --user <UID>  effective user id or name
  -U, --User <UID>     real user id or name

 The selection options take as their argument either:
   a comma-separated list e.g. '-u root,nobody' or
   a blank-separated list e.g. '-p 123 4567'

Output formats:
  -F                   extra full
  -f                   full-format, including command lines
   f, --forest         ascii art process tree
  -H                   show process hierarchy
  -j                   jobs format
   j                   BSD job control format
  -l                   long format
   l                   BSD long format
  -M, Z                add security data (for SELinux)
  -O <format>          preloaded with default columns
   O <format>          as -O, with BSD personality
  -o, o, --format <format>
                       user-defined format
   s                   signal format
   u                   user-oriented format
   v                   virtual memory format
   X                   register format
  -y                   do not show flags, show rss vs. addr (used with -l)
      --context        display security context (for SELinux)
      --headers        repeat header lines, one per page
      --no-headers     do not print header at all
      --cols, --columns, --width <num>
                       set screen width
      --rows, --lines <num>
                       set screen height

Show threads:
   H                   as if they were processes
  -L                   possibly with LWP and NLWP columns
  -m, m                after processes
  -T                   possibly with SPID column

Miscellaneous options:
  -c                   show scheduling class with -l option
   c                   show true command name
   e                   show the environment after command
   k,    --sort        specify sort order as: [+|-]key[,[+|-]key[,...]]
   L                   show format specifiers
   n                   display numeric uid and wchan
   S,    --cumulative  include some dead child process data
  -y                   do not show flags, show rss (only with -l)
  -V, V, --version     display version information and exit
  -w, w                unlimited output width

         --help <simple|list|output|threads|misc|all>
                       display help and exit
```

例如显示所有进程信息：
```
ps -ef
```
会看到类似信息
```
UID        PID  PPID  C STIME TTY          TIME CMD
root         1     0  0 Feb13 ?        00:00:01 /sbin/init splash
root         2     0  0 Feb13 ?        00:00:00 [kthreadd]
root         3     2  0 Feb13 ?        00:00:00 [ksoftirqd/0]
-----------
UID：    进程属于那个用户
PID：    进程 ID
PPID：   进程父 ID
C：      CPU 使用资源的百分比
STIME：  进程启动时间
TTY：    登录终端
TIME：   用掉的 CPU 时间
CMD：    指令
```
列出所有正在内存中的程序
```
ps aux
```
会看到类似信息
```
USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root         1  0.0  0.0 120068  3472 ?        Ss   Feb13   0:01 /sbin/init splash
root         2  0.0  0.0      0     0 ?        S    Feb13   0:00 [kthreadd]
root         3  0.0  0.0      0     0 ?        S    Feb13   0:00 [ksoftirqd/0]
root         5  0.0  0.0      0     0 ?        S<   Feb13   0:00 [kworker/0:0H]
-----------
USER：     该 process 属于那个使用者账号的
PID：      该 process 的号码
%CPU：     该 process 使用掉的 CPU 资源百分比
%MEM：     该 process 所占用的物理内存百分比
VSZ：      该 process 使用掉的虚拟内存量 (Kbytes)
RSS：      该 process 占用的固定的内存量 (Kbytes)
TTY：      该 process 是在那个终端机上面运作，若与终端机无关，则显示 ?，另外， tty1-tty6 是本机上面的登入者程序，若为 pts/0 等等的，则表示为由网络连接进主机的程序。
STAT：     该程序目前的状态，主要的状态有
R：        该程序目前正在运作，或者是可被运作
S：        该程序目前正在睡眠当中 (可说是 idle 状态)，但可被某些讯号 (signal) 唤醒。
T：        该程序目前正在侦测或者是停止了
Z：        该程序应该已经终止，但是其父程序却无法正常的终止他，造成 zombie (疆尸) 程序的状态
START：    该 process 被触发启动的时间
TIME：     该 process 实际使用 CPU 运作的时间
COMMAND：  该程序的实际指令

```
