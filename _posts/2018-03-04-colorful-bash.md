---
layout: post
title:  "设置bash颜色"
categories: linux bash
---

在使用bash时默认是白底黑字，bash是提供了彩色显示的能力（ANSI/VT100）, 使用语法`<Esc>[FormatCodem`可以设置字符和背景的颜色。其中FormatCode需要替换成样式参数，[Esc]可以使用"\e", "\033", "\x1B"三种的一种。如`echo -e "\e[1;31;42m Yes it is awful \e[0m"`显示的为"粗体 + 红色字体 + 绿色背景", "\e[0m"表示重置所有颜色设置，进行性出现在需要设置颜色字符的末尾。


- 使用sed高亮显示字符，
```
$ echo "this is root user" | sed -e 's/root/\x1b[1;31m&\x1b[0m/' # \x1b
$ echo "this is root user" | sed -e 's/root/\o033[1;31m&\o033[0m/'# \o033
```

- grep高亮显示字符,使用"--color"选项
```
$ grep --color root /etc/passwd
```


#### 引用
1. [https://misc.flogisoft.com/bash/tip_colors_and_formatting](https://misc.flogisoft.com/bash/tip_colors_and_formatting)
