# tree 命令

## 1. 语法

`tree  [-acdfghilnpqrstuvxACDFJQNSUX] [-L level [-R]] [-H baseHREF] [-T title] [-o filename] [-P pattern] [-I pattern] [--gitignore] [--matchdirs] [--metafirst] [--ignore-case] [--nolinks] [--inodes] [--device] [--sort[=]name] [--dirsfirst] [--filesfirst] [--filelimit #] [--si] [--du] [--prune] [--timefmt[=]format] [--fromfile] [--info]  [--noreport] [--version] [--help] [--] [directory ...]`

## 2. 常用选项

|选项|含义|
|:-|:-|
|`-a`|显示隐藏文件|
|`-o filename`|指定输出文件|

## 3. 常用功能

`tree -a [directory]`：递归显示当前（或指定）目录下的所有文件