---
title: 每天一个 Linux 命令（1）： useradd
date: 2017-02-08 11:46:53
tags:
 - Linux
category: 
 - Linux
thumbnail: 
author: bsyonline
lede: "没有摘要"
---

在伯乐在线上看到一个《每天一个 Linux 命令》的系列文章，觉得有必要对自己用过的和没有用过得命令做一个归纳，只是简单的 How-To 类的说明，算是对有些很久不用的知识做个备忘。
<!-- more -->
Linux 的命令有很多，常用的有 100 个？，所以也没有什么顺序，想到哪写到哪。今天用到了一个创建用户命令 useradd 。
### useradd

useradd 命令的作用是创建新用户或更新默认用户。

useradd 命令格式：
```
Usage: useradd [options] LOGIN

Options:
  -b, --base-dir BASE_DIR       base directory for the home directory of the new account
  -c, --comment COMMENT         GECOS field of the new account
  -d, --home-dir HOME_DIR       home directory of the new account
  -D, --defaults                print or change default useradd configuration
  -e, --expiredate EXPIRE_DATE  expiration date of the new account
  -f, --inactive INACTIVE       password inactivity period of the new account
  -g, --gid GROUP               name or ID of the primary group of the new account
  -G, --groups GROUPS           list of supplementary groups of the new account
  -h, --help                    display this help message and exit
  -k, --skel SKEL_DIR           use this alternative skeleton directory
  -K, --key KEY=VALUE           override /etc/login.defs defaults
  -l, --no-log-init             do not add the user to the lastlog and faillog databases
  -m, --create-home             create the user's home directory
  -M, --no-create-home          do not create the user's home directory
  -N, --no-user-group           do not create a group with the same name as the user
  -o, --non-unique              allow to create users with duplicate (non-unique) UID
  -p, --password PASSWORD       encrypted password of the new account
  -r, --system                  create a system account
  -R, --root CHROOT_DIR         directory to chroot into
  -s, --shell SHELL             login shell of the new account
  -u, --uid UID                 user ID of the new account
  -U, --user-group              create a group with the same name as the user
  -Z, --selinux-user SEUSER     use a specific SEUSER for the SELinux user mapping
      --extrausers              Use the extra users database
```

实际上用过的选项只有 -G 和 -g ， -g 是指定用户所属的主组， -G 是指定用户的附加组。
如创建一个用户 rolex ， 主组是 hadoop ，副组是 ops  
```
useradd -g hadoop -G ops rolex
```
指定的组必须是存在的，所以这个命令通常和 groupadd 和 passwd 一起使用。
### groupadd

groupadd 命令格式：
```
Usage: groupadd [options] GROUP
-f, --force                   exit successfully if the group already exists, and cancel -g if the GID is already used
-g, --gid GID                 use GID for the new group
-h, --help                    display this help message and exit
-K, --key KEY=VALUE           override /etc/login.defs defaults
-o, --non-unique              allow to create groups with duplicate (non-unique) GID
-p, --password PASSWORD       use this encrypted password for the new group
-r, --system                  create a system account
-R, --root CHROOT_DIR         directory to chroot into
    --extrausers              Use the extra users database

```
### passwd

passwd 命令格式：
```
Usage: passwd [options] [LOGIN]

Options:
  -a, --all                     report password status on all accounts
  -d, --delete                  delete the password for the named account
  -e, --expire                  force expire the password for the named account
  -h, --help                    display this help message and exit
  -k, --keep-tokens             change password only if expired
  -i, --inactive INACTIVE       set password inactive after expiration to INACTIVE
  -l, --lock                    lock the password of the named account
  -n, --mindays MIN_DAYS        set minimum number of days before password change to MIN_DAYS
  -q, --quiet                   quiet mode
  -r, --repository REPOSITORY   change password in REPOSITORY repository
  -R, --root CHROOT_DIR         directory to chroot into
  -S, --status                  report password status on the named account
  -u, --unlock                  unlock the password of the named account
  -w, --warndays WARN_DAYS      set expiration warning days to WARN_DAYS
  -x, --maxdays MAX_DAYS        set maximum number of days before password
```

### 用户和组的相关知识

创建的用户可以在 /etc/passwd 中查看
```
rolex@T450:~$ cat /etc/passwd
root:x:0:0:root:/root:/bin/bash
rolex:x:1000:1000:rolex,,,:/home/rolex:/bin/bash
```
格式为：
```
用户名:口令:用户标识号:组标识号:注释性描述:主目录:登录 Shell

1. 用户名：
2. 口令：/etc/passwd 是对所有人可见的，所以口令用 x 作为站位，真正的口令存放在 /etc/shadow 中。
   /etc/passwd 和 /etc/shadow 是一一对应的，通过 pwconv 程序生成。
3. 用户标识号：用户在系统中的唯一标识，root 用户 为 0 。
4. 组标识号：对应着 /etc/group 文件中的一条记录。格式为：``组名:口令:组标识号:组内用户列表``
5. 注释性描述：真的就是注释而已。
6. 主目录：用户的起始工作目录，就是登录后所处的位置。
7. 登录 Shell：Shell 是用户与 Linux 系统之间的接口，shell 程序有很多种类，Ubuntu 默认是 /bin/bash 。
```
