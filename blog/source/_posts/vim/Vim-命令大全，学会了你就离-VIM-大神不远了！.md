---
title: Vim 命令大全，学会了你就离 VIM 大神不远了！
tags:
  - vim
abbrlink: 9a404bfa
date: 2024-10-12 09:22:44
---

​	日常开发过程中，免不了需要键盘、鼠标（触摸板）来回切换，本人有点懒哈哈哈！想着尝试学习 VIM，这样在开发过程中就可以双手不离键盘就可以操控一切了，这种感觉想想都是太棒了！期待早日熟悉 VIM 命令操作。

​	对于未使用过  VIM 的朋友来说，可能无法体会到这种感觉。由于学习使用 VIM 有一定的学习成本，只有做到非常熟练的程度才能感受到它带来的快捷。

​	这里我就记录日常过程中有使用过的 VIM 命令，建议有需要学习 VIM 的朋友，可以按照文章进行匹配搜索想要查看的命令。记得要多多尝试哦，加油！！！



### 1. VIM 模式

​	VIM 模式分为三种，**正常模式(NORMAL)**、**插入模式(INSERT)**、**可视模式(VISUAL)**。

> **正常模式（NORMAL）**：按`ESC`或`Ctrl+[`进入，左下角显示文件名或为空。
>
> **插入模式（INSERT）**：按`i`进入，左下角显示`--INSERT--`
>
> **可视模式（VISUAL）**：按`v`进入，左下角显示`--VISUAL--`

### 2. 打开文件

```shell
# 打开单个文件
vim file
# 同时打开多个文件
vim file1 file2

# 在 Vim 窗口中打开一个新文件
:open file

【示例】
# 当前打开了 test1，做了一些编辑没保存
:open!	# 放弃这些修改，并重新打开文件

# 以只读形式打开文件，但是仍然可以使用 :wq! 写入
vim -R file

# 强制性关闭修改功能，无法使用 :wq! 写入
vim -M file
```

### 3. 插入命令

进入插入模块有多种方式，`i`、`I`、`a`、`A`、`o`、`O`。

> **i**：在当前位置前插入
>
> **I**：在当前位置行首插入
>
> **a**：在当前位置后插入
>
> **A**：在当前位置行尾插入
>
> **o**：在当前行之后插入一行
>
> **O**：在当前行之前插入一行

### 4. 查找命令

#### 简单查找

> **/text**：查找 text，按`n`键查找下一个，按`N`键查找前一个。
>
> **?text**：查找 text，反向查找，按`n`键查找下一个，按`N`键查找前一个
>
> :set ignorecase	# 忽略大小写的查找
>
> :set noignorecase	# 不忽略大小写的查找

#### 快速查找

> `*`：向后（下）寻找游标所在处的单词
>
> `#`：向前（上）寻找游标所在处的单词

#### 精准查找：匹配单词查找

如果文本中有如下三个单词

`hello`、`helloworld`、`helloJava`

那按照正常的 `/hello`方式查找，这三个单词都能匹配到。

可以按如下方式进行精准查找

```shell
/hello\>
```

#### 精准查找：匹配行首、行末

```shell
# hello 位于行首
/^hello

# world 位于行末
/world$
```

### 5. 替换命令

> `~`：反转游标字母大小写
>
> `r<字母>`：将当前字符替换为所写字母
>
> `R<字母><字母>`：连续替换字母
>
> `cc`：替换整行（删除当前行，并在下一行插入）
>
> `cw`：替换一个单词（删除一个单词，进入插入模式），前提是游标处于单词第一个字母（用`b`定位到单词最前面）
>
> `C`：（大写 C）替换至行尾，删除游标到行尾的内容
>
> 
>
> `:s/old/new`：用 old 替换 new，替换**当前行**的第一个匹配
>
> `:s/old/new/g`：用 old 替换 new，替换**当前行**的所有匹配
>
> `:%s/old/new`：用 old 替换 new，替换**所有行**的第一个匹配
>
> `:%s/old/new/g`：用 old 替换 new，替换**整个文件**的所有匹配
>
> `:1,5 s/^/	/g`：在第 1 行至第 5 行每行前面加一个空格，用于缩进。
>
> `ddp`：交换光标所在行和其下紧邻的一行。`dd`会删除当前行并复制，`p`粘贴

### 6. 撤销和重做

> `u`：撤销（Undo）
>
> `U`：撤销对整行的操作
>
> `Ctrl + r`：反撤销

### 7. 删除命令

​	Vim 其实没有单纯的删除命令，你可以理解为**剪切**更加准确。

**以字符为单位删除**

> `x`：删除当前字符
>
> `5x`：删除当前字符 5 次
>
> `X`：删除当前字符的前一个字符
>
> `5X`：删除当前光标向前 5 个字符
>
> `dl`：删除当前字符，dl=x
>
> `dh`：删除前一个字符，dh=X
>
> `D`：删除当前字符至行尾，D=d$
>
> `d$`：删除当前字符至**行尾**
>
> `d^`：删除当前字符之前至**行首**所有内容

**以单词为单位删除**

> `dw`：删除当前字符到单词尾
>
> `daw`：删除当前字符所在单词

**以行为单位删除**

> `dd`：删除当前行
>
> `dj`：删除下一行
>
> `dk`：删除上一行
>
> 
>
> `dgg`：删除当前行至文档首部
>
> `d1G`：删除当前行至文档首部
>
> `dG`：删除当前行至文档尾部
>
> `10d`：删除当前行开始的 10 行
>
> `:1,10d`：删除 1- 10 行
>
> `10,$d`：删除第 10 行及之后所有行
>
> `1,$d`：删除所有行
>
> `J`：合并两行

### 8. 复制粘贴

在普通模式下使用`y`复制

> `yy`：复制游标所在的整行
>
> `y^`：复制至行首，或`y0`，不包含光标所在处字符
>
> `y$`：复制至行尾。含光标所在处字符
>
> `yw`：复制一个单词
>
> `y2w`：复制两个单词
>
> `yG`：复制至文本末
>
> `y1G`：复制至文件开头

普通模式中使用`p`粘贴

> `p`：代表粘贴至光标后（下边，右边）
>
> `P`：代表粘贴至光标前（上边、左边）

### 9. 剪切粘贴

> `dd`：剪切命令，剪切当前行
>
> `ddp`：剪切当前行并粘贴，可实现当前行与下一行调换位置

### 10. 退出保存

> `:wq`：保存并退出
>
> `ZZ`：保存并退出
>
> `:q!`：强制退出并放弃所有更改
>
> `:e!`：放弃所有修改，并重新打开原来文件
>
> `:sav(eas) new.txt`：另存为一个新文件，退出原文件的编辑且不会保存
>
> `:f(ile) new.txt`：新开一个文件，并不会保存，退出原文件的编辑且不会保存

### 11. 移动命令

**上下左右**

> `hjkl`：左、下、上、右移动一个字符

**以单词为单位移动**

> `w`：向前移动一个单词（光标停留在单词首部）
>
> `b`：向后移动一个单词（光标停留在单词首部）
>
> `e`：向前移动一个单词（光标停留在单词尾部）
>
> `ge`：向后移动一个单词（光标停留在单词尾部）

**以句为单位移动**

> `(`：移动到句首
>
> `)`：移动到句尾

**跳转到文件的首尾**

> `gg`：移动到文件头
>
> `G`：移动到文件尾

**其他移动方法**

> `^`：移动到本行第一个非空白字符上
>
> `0`：移动到本行第一个字符上（可以是空格）

### 12. 排版功能

**缩进**

> `:set shiftwidth?`：查看缩进值
>
> `:set shiftwidth=4`：设置缩进值为 4
>
> `>>`：向右缩进
>
> `<<`：向左缩进

### 13. 注释命令

**多行注释**

> 进入命令行模式，按`ctrl + v`进入 `VISUAL BLOCK`模式，然后按`j`或`k`选中多行
>
> 按大写字母`I`，再插入注释符`//`
>
> 按`ESC`键就会把所有选中的行注释了

**取消多行注释**

> 进入命令行模式，按`ctrl + v`进入`VISUAL BLOCK`模式，按字母`l`横向选中列的个数
>
> 按字母`j`或`k`选中注释符合
>
> 按`d`键就可以取消全部注释了

**复杂注释**

> `:1,3 s/^/#/g`：注释第1 - 3 行
>
> `:1,3 s/^#//g`：取消第1 - 3 行的注释
>
> 
>
> `:1,$ s/^/#/g`：注释整个文档
>
> `:1,$ s/^$//g`：取消整个文档注释
>
> 
>
> `:%s/^/#/g`：注释整个文档
>
> `%s/^#//g`：取消注释整个文档

### 14. 调整视野查看

> `zz`：把当前行置为屏幕正中央
>
> `zt`：把当前行置为屏幕顶端
>
> `zb`：把当前行置为屏幕底端
>
> 
>
> `ctrl + e`：向下滚动一行
>
> `ctrl + y`：向上滚动一行
>
> 
>
> `ctrl + d`：向下滚动半屏
>
> `ctrl + u`：向上滚动半屏
>
> 
>
> `ctrl + f`：向下滚动一屏
>
> `ctrl + b`：向上滚动一屏
