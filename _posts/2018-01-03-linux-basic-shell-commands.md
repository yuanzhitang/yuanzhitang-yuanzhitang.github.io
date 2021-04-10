---
layout: post
title:  "Linux 命令：文件目录操作与实例"
date:   2018-01-03 13:48:08
categories: Linux
tags: Ubuntu Linux Shell
author: Michael
mathjax: true
---

* content
{:toc}

本文介绍基础的文件操作：创建，移动，编辑，删除 文件和文件夹



## 命令与案例

### mkdir 创建目录

--创建两个目录
```shell
tangym@ubuntu:~$ mkdir test2 test3
```

--在test1下面创建一个新的目录mydir
```shell
tangym@ubuntu:~$ mkdir test1/mydir
```

--尝试在test100下面创建一个新的目录mydir，但不成功，因为test100这个目录不存在
```shell
tangym@ubuntu:~$ mkdir test100/mydir 
mkdir: cannot create directory `test100/mydir': No such file or directory
```

-- 强制创建父子这两个文件, 尽管test100这个父目录不存在
```shell
tangym@ubuntu:~$ mkdir -p test100/mydir
```

### touch 创建文件



--创建hello文件在当前目录
```shell
tangym@ubuntu:~$ touch hello 
echo
```

-- 写 "hello" 到这个目录
```shell
tangym@ubuntu:~/test1$ cat hellobackup
tangym@ubuntu:~/test1$ echo "hello" > hellobackup 
tangym@ubuntu:~/test1$ cat hellobackup
hello
```



### mv 移动或重命名文件



-- 移动文件 hello到test1文件夹
```shell
tangym@ubuntu:~$ mv hello test1
```

--重命名文件hello为hellobackup
```shell
tangym@ubuntu:~/test1$ mv hello hellobackup
```

### cp 拷贝文件
```shell
tangym@ubuntu:~$ cp pse2 test2 -- copy file pse2 to test2 folder
```

### rm/rmdir 删除文件和文件夹



--删除文件hello
```shell
tangym@ubuntu:~$ rm hello
```

--删除文件夹test2
```shell
tangym@ubuntu:~$ rmdir test2
```


### 输入重定向至文件:



下面将会把界面的输入写入文件hellobackup文件
```shell
tangym@ubuntu:~$ cat <<EOF >hellobackup
> hello world!
> real func
> EOF
```
常看文件内容
```shell
tangym@ubuntu:~$ cat hellobackup
hello world!
real func
tangym@ubuntu:~$
```

### 完整的例子（创建和删除文件）
```shell
tangym@ubuntu:~$ cd mhydir
tangym@ubuntu:~/mhydir$ ls
tangym@ubuntu:~/mhydir$ touch test
tangym@ubuntu:~/mhydir$ ls
test
tangym@ubuntu:~/mhydir$ rm test
tangym@ubuntu:~/mhydir$ ls
tangym@ubuntu:~/mhydir$ touch test
tangym@ubuntu:~/mhydir$ rm -i test   --Will Confirm whether delete the file
rm: remove regular empty file `test'? n
tangym@ubuntu:~/mhydir$ ls
test
tangym@ubuntu:~/mhydir$ rm -i test
rm: remove regular empty file `test'? y
tangym@ubuntu:~/mhydir$ ls
tangym@ubuntu:~/mhydir$
```