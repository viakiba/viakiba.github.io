---
layout: post
title: 2020-07-17-Linux 命令之GREP
categories: [LINUX]
description: Linux
keywords: LINUX 命令之GREP
---

### 说明
linux 的 grep 命令用于查找文件里符合条件的字符串。

    grep [OPTION]... PATTERN [FILE]...
    grep -E 等同于 egerp
    grep -F 等同于 fgerp

详细参数 通过 'grep -help' 获取

#### 参数说明

```
模式选择和解释：
  -E, --extended-regexp     PATTERN是扩展的正则表达式
  -F, --fixed-strings       PATTERN是一组用换行符分隔的字符串
  -G, --basic-regexp        PATTERN是基本的正则表达式（默认）
  -P, --perl-regexp         PATTERN是Perl正则表达式
  -e, --regexp=PATTERN      使用PATTERN进行匹配
  -f, --file=FILE           从文件获取模式
  -i, --ignore-case         忽略大小写区别
  -w, --word-regexp         强制PATTERN只匹配整个单词
  -x, --line-regexp         强制PATTERN只匹配整行
  -z, --null-data           数据行以0字节结尾，而不是换行符

输出控制:
  -m, --max-count=NUM       在NUM条选定的行之后停止
  -b, --byte-offset         用输出行打印字节偏移
  -n, --line-number         带输出行的打印行号
      --line-buffered       刷新每一行的输出
  -H, --with-filename       使用输出行打印文件名
  -h, --no-filename         在输出中取消文件名前缀
      --label=LABEL         使用LABEL作为标准输入文件名前缀
  -o, --only-matching       仅显示匹配PATTERN的线的一部分
  -q, --quiet, --silent     禁止所有正常输出
      --binary-files=TYPE   假定二进制文件是TYPE；
                            TYPE是“二进制”，“文本”或“不匹配”
  -a, --text                等效于--binary-files = text
  -I                        等效于--binary-files = without-match
  -d, --directories=ACTION  如何处理目录；
                            ACTION是“读取”，“递归”或“跳过”
  -D, --devices=ACTION      如何处理设备，FIFO和套接字；
                            动作是“读取”或“跳过”
  -r, --recursive           像--directories = recurse
  -R, --dereference-recursive  同样，但请遵循所有符号链接
      --include=FILE_PATTERN  只搜索与FILE模式匹配的文件
      --exclude=FILE_PATTERN  跳过与FILE模式匹配的文件和目录
      --exclude-from=FILE   跳过与FILE模式匹配的文件和目录
      --exclude-dir=PATTERN  匹配模式的目录将被跳过。
  -L, --files-without-match 仅打印没有选定行的文件名
  -l, --files-with-matches  仅打印具有选定行的文件名
  -c, --count               每个文件仅打印选定行的数量
  -T, --initial-tab         使标签排成一行（如果需要）
  -Z, --null                在文件名后打印0字节

上下文控制:
  -B, --before-context=NUM  打印前导上下文的NUM行
  -A, --after-context=NUM   打印NUM行尾随上下文
  -C, --context=NUM         打印NUM行输出上下文
  -NUM                      与--context = NUM​​相同
      --color[=WHEN],
      --colour[=WHEN]       使用标记突出显示匹配的字符串；
                            何时“始终”，“从不”或“自动”
  -U, --binary              不要在EOL（MSDOS /Windows）剥离CR字符

附属参数:
  -s, --no-messages         禁止显示错误消息
  -v, --invert-match        选择不匹配的行
  -V, --version             显示版本信息并退出
      --help                显示此帮助文本并退出
```

### 实践

#### 准备一个测试文件

test.txt 与 test1.txt 文件 内容一致

```txt
1234567890
123123123123
123ABC23
abc123
````

#### 实践

基本模式 主要和输出控制参数搭配使用
```shell
指定文件指定字符
$ grep ABC test.txt
123ABC23

通配文件指定字符
$ grep ABC test*
test.txt:123ABC23
test1.txt:123ABC23

递归文件夹指定字符
$ grep -R ABC ./
./test.txt:123ABC23
./test1.txt:123ABC23

通过"-v"参数可以打印出不符合条件行的内容
$ grep -v ABC test.txt
1234567890
123123123123
abc123

通过 -n 参数打印行号
$ grep -n ABC test*
test.txt:3:123ABC23
test1.txt:3:123ABC23

-n -R 混合使用
$ grep -R -n ABC ./
./test.txt:3:123ABC23
./test1.txt:3:123ABC23

-i 不区分大小写
$ grep -n -i ABC test.txt
3:123ABC23
4:abc123

仅打印具有选定行的文件名
$ grep -l ABC test*
test.txt
test1.txt

```

#### 正则模式

```shell
匹配开头以数字开始的行
$ grep '^[0-9]' test.txt
1234567890
123123123123
123ABC23

匹配开头以字母开始的行
$ grep '^[a-zA-Z]' test.txt
abc123

匹配开头以ab开始的行
$ egrep '^ab' test.txt
abc123
```

