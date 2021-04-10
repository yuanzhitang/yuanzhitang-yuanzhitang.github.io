---
layout: post
title:  "Linux 常用命令与案例"
date:   2016-05-25 12:52:10
categories: Linux
tags: Ubuntu Linux Shell
author: Michael
mathjax: true
---

* content
{:toc}

本文介绍基础的Linux Shell命令和案例：


cd /  : entry root directory
```shell
tangym@ubuntu:/etc$ cd /
tangym@ubuntu:/$ pwd
```
cd ~  : entry user root directory
```shell
tangym@ubuntu:/$ cd ~
tangym@ubuntu:~$ pwd
/home/tangym
```
ls    : list all files.
pwd   : show current directory
cd /home :  entry home directory
cd .. : entry the last parent directory

cat fstab: show the text file content.

cat fs， then press <Tab>. Shell will fill the whole file name automtically.
ls m*  : list all the file name starts with: m
ls magi?  : list all the file name starts with : magi and following one character.
          Like: magic
ls m*[od]  : list all starts with : m, end with: o or d.
ls -l   : show all file's properties:
-------------------------
```shell
tangym@ubuntu:~$ ls -l
total 204112
drwxr-xr-x  2 tangym tangym      4096 May 29  2015 Desktop
drwxr-xr-x  2 tangym tangym      4096 May 29  2015 Documents
drwxr-xr-x  3 tangym tangym      4096 May 14 08:53 Downloads
-rw-r--r--  1 tangym tangym      8445 May 29  2015 examples.desktop
-rw-rw-r--  1 tangym tangym     10240 Jan 20  2011 ez_setup.py
drwxr-xr-x  2 tangym tangym      4096 May 29  2015 Music
drwxr-xr-x  2 tangym tangym      4096 May 29  2015 Pictures
drwxr-xr-x  2 tangym tangym      4096 May 29  2015 Public

```
dir : show all files
vdir: show all files and its properties

cat -n .bashrc: show file content with rowcount
```shell
tangym@ubuntu:~$ cat -n .bashrc
     1 # ~/.bashrc: executed by bash(1) for non-login shells.
     2 # see /usr/share/doc/bash/examples/startup-files (in the package bash-doc)
     3 # for examples
     4
```
more .bashrc : show file content in separate page.
```shell
tangym@ubuntu:~$ more .bashrc
# ~/.bashrc: executed by bash(1) for non-login shells.
# see /usr/share/doc/bash/examples/startup-files (in the package bash-doc)
# for examples
```
head -n 5 .bashrc : show first 5 row content
tail -n 5 .bashrc    : show last 5 row content.

grep PS1 .bashrc    : find all row content with: PS1
```shell
tangym@ubuntu:~$ grep PS1 .bashrc
[ -z "$PS1" ] && return
    PS1='${debian_chroot:+($debian_chroot)}\[\033[01;32m\]\u@\h\[\033[00m\]:\[\033[01;34m\]\w\[\033[00m\]\$ '
    PS1='${debian_chroot:+($debian_chroot)}\u@\h:\w\$ '
    PS1="\[\e]0;${debian_chroot:+($debian_chro
```
find ./ -name *bash*  : Find file name with: bash in current directory.
```shell
tangym@ubuntu:~$ find ./ -name *bash*
./.bash_history
./.bashrc
./.bash_logout
./redis-py-master/vagrant/.bash_profile
./Downloads/redis-py-master/vagrant/.bash_profile
tangym@ubuntu:~$ locate bash  // similar to find . find file/directory with :back
tangym@ubuntu:~$ sudo updatedb //force update local db for Locate command
tangym@ubuntu:~$ firefox www.baidu.com : open firefox webbrowser
```
whereis find  : get command's notebook
```shell
tangym@ubuntu:~$ whereis find : get notebook
find: /usr/bin/find /usr/bin/X11/find /usr/share/man/man1/find.1.gz
tangym@ubuntu:~$ whereis -b find 
```
find: /usr/bin/find /usr/bin/X11/find

which ls: find where is ls command

```shell
tangym@ubuntu:~$ which ls
/bin/ls
```

who
whoami
uname
uname -a
uname -r : kernel version

```shell
tangym@ubuntu:~$ who
tangym   tty7         2016-05-23 23:50
tangym   pts/1        2016-05-23 23:50 (:0)
tangym   pts/2        2016-05-23 23:58 (:0)
tangym   pts/3        2016-05-23 23:59 (:0)
tangym   pts/4        2016-05-24 19:49 (:0)
tangym@ubuntu:~$ who am i
tangym   pts/4        2016-05-24 19:49 (:0)
tangym@ubuntu:~$ whoami
tangym
tangym@ubuntu:~$ uname
Linux
tangym@ubuntu:~$ uname -a
Linux ubuntu 3.2.0-29-generic #46-Ubuntu SMP Fri Jul 27 17:03:23 UTC 2012 x86_64 x86_64 x86_64 GNU/Linux
```

man find  :　get the find command decription

```shell
tangym@ubuntu:~$ man find
```



whatis ls: get the command common usage.
```shell
tangym@ubuntu:~$ whatis ls
ls (1)               - list directory contents
```