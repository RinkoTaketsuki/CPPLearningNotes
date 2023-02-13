# Shell 使用手册

> 示例代码运行环境：阿里云 ECS，Ubuntu 22.04

查看命令的搜索路径：

```sh
ecs-user@ALIYUN-ECS:~$ echo $PATH
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin
```

查看默认 shell（登录 shell、开机启动的 shell）：

```sh
ecs-user@ALIYUN-ECS:~$ echo $SHELL
/bin/bash
```

查看历史记录：

```sh
ecs-user@ALIYUN-ECS:~$ history
    1  ls
    2  ls -al
    3  sudo apt update
    4  sudo apt upgrade
    5  hostnamectl
    6  vi /etc/sysconfig/network
    7  uname -n
    8  cd /etc
    9  ls
   10  cd sysctl.d
   11  ls
   12  sudo vim 10-network-security.conf 
   13  sudo vim 10-kernel-hardening.conf 
   14  cat hostname
```

查看当前工作目录的绝对路径：

```sh
# pwd 命令添加选项 -P 可以将软链接替换为真实路径
ecs-user@ALIYUN-ECS:~$ pwd
/home/ecs-user
ecs-user@ALIYUN-ECS:~$ cd bin
ecs-user@ALIYUN-ECS:/bin$ pwd
/bin
ecs-user@ALIYUN-ECS:/bin$ pwd -P
/usr/bin
ecs-user@ALIYUN-ECS:/bin$ echo $PWD
/bin
```

查看家目录：

```sh
ecs-user@ALIYUN-ECS:~$ echo ~
/home/ecs-user
ecs-user@ALIYUN-ECS:~$ echo $HOME
/home/ecs-user
```

查看命令所在目录：

```sh
ecs-user@ALIYUN-ECS:~$ which echo
/usr/bin/echo
```

创建多级目录：

```sh
ecs-user@ALIYUN-ECS:~$ mkdir a/b/c
mkdir: cannot create directory ‘a/b/c’: No such file or directory
ecs-user@ALIYUN-ECS:~$ mkdir -p a/b/c
ecs-user@ALIYUN-ECS:~$ tree
.
└── a
    └── b
        └── c

3 directories, 0 files
```

快捷键：

<kbd>Tab</kbd>：命令补全，参数补全，文件名补全

<kbd>Ctrl</kbd> + <kbd>P</kbd>：同 <kbd>↑</kbd>，使用上一个命令

<kbd>Ctrl</kbd> + <kbd>N</kbd>：同 <kbd>↓</kbd>，使用下一个命令

<kbd>Ctrl</kbd> + <kbd>B</kbd>：同 <kbd>←</kbd>，向左移动光标

<kbd>Ctrl</kbd> + <kbd>F</kbd>：同 <kbd>→</kbd>，向右移动光标

<kbd>Ctrl</kbd> + <kbd>A</kbd>：同 <kbd>Home</kbd>，移动光标到头部

<kbd>Ctrl</kbd> + <kbd>E</kbd>：同 <kbd>End</kbd>，移动光标到尾部

<kbd>Ctrl</kbd> + <kbd>H</kbd>：同 <kbd>Backspace</kbd>，删除光标前面的一个字符

<kbd>Ctrl</kbd> + <kbd>D</kbd>：同 <kbd>Delete</kbd>，删除光标后面的一个字符

> 光标后面的定义：高亮方块选中的位置（包括）之后，或竖线右边的位置

<kbd>Ctrl</kbd> + <kbd>U</kbd>：删除光标前面的所有字符

<kbd>Ctrl</kbd> + <kbd>K</kbd>：删除光标后面的所有字符
