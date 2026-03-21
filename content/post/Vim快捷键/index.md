---
title: Vim快捷键
description: ""
slug: 2022-02-21-Vim快捷键
date: 2026-02-21
image: vim.png
categories:
  - 运维&云原生
tags:
  - vim
  - Linux
---
## 全局

-   :help keyword - 打开关键字帮助
-   :saveas file - 另存为
-   :close - 关闭当前窗口
-   K - open man page for word under the cursor

## 光标

-   h - 左移光标
-   j - 下移光标
-   k - 上移光标
-   l - 右移光标
-   H - 移动到当前页面顶部
-   M - 移动到当前页面中间
-   L - 移动到当前页面底部
-   w - 移动到下个单词开头
-   W - 移动到下个单词开头(单词含标点)
-   e - 移动到下个单词结尾
-   E - 移动到下个单词结尾(单词含标点)
-   b - 移动到上个单词结尾
-   B - 移动到上个单词结尾(单词含标点)
-   % - move to matching character (default supported pairs: '()', '{}', '\[\]' - use `:h matchpairs` in vim for more info)
-   0 - 移动到行首
-   ^ - 移动到行首的非空白符
-   $ - 移动到行尾
-   g\_ - 移动到行内最后一个非空白符
-   gg - 移动到文件第一行
-   G - 移动到文件最后一行
-   5G - 移动到第五行
-   fx - 移动到字符 x 下次出现的位置
-   tx - 移动到字符 x 下次出现的位置的前一个字符
-   Fx - jump to previous occurence of character x
-   Tx - jump to after previous occurence of character x
-   ; - repeat previous f, t, F or T movement
-   , - repeat previous f, t, F or T movement, backwards
-   } - 移动到下一个段落 (当编辑代码时则为函数／代码块)
-   { - 移动到上一个段落 (当编辑代码时则为函数／代码块)
-   zz - 移动屏幕使光标居中
-   Ctrl + e - move screen down one line (without moving cursor)
-   Ctrl + y - move screen up one line (without moving cursor)
-   Ctrl + b - 向后滚动一屏
-   Ctrl + f - 向前滚动一屏
-   Ctrl + d - 向前滚动半屏
-   Ctrl + u - 向后滚动半屏

Tip 命令前追加数字表示命令的重复次数, 比如 4j 表示向下移动四行

## 插入模式 - 插入/追加文本

-   i - 从光标前开始插入字符
-   I - 从行首开始插入字符
-   a - 从光标后开始插入字符
-   A - 从行尾开始插入字符
-   o - 在当前行之下另起一行, 开始插入字符
-   O - 在当前行之上另起一行, 开始插入字符
-   ea - 从当前单词末尾开始插入
-   Esc - 退出插入模式

## 编辑

-   r - 替换当前字符
-   J - 将下一行合并到当前行
-   gJ - join line below to the current one without space in between
-   gwip - reflow paragraph
-   cc - 清空当前行, 然后进入插入模式
-   c$ - 从光标位置开始, 修改当前行
-   ciw - change (replace) entire word
-   cw - 从光标位置开始, 修改单词
-   s - 删除当前字符, 然后进入插入模式
-   S - 清空当前行, 然后进入插入模式 (同cc)
-   xp - 当前字符后移
-   u - 撤销
-   Ctrl + r - 重复
-   . - 再次执行上个命令

## 选择文本（可视化模式）

-   v - 进入可视化模式, 移动光标高亮选择, 然后可以对选择的文本执行命令(比如y-复制)
-   V - 进入可视化模式(行粒度选择)
-   o - 切换光标到选择区开头/结尾
-   Ctrl + v - 进入可视化模式(矩阵选择)
-   O - 切换光标到选择区的角
-   aw - 选择当前单词
-   ab - 选择被 () 包裹的区域(含括号)
-   aB - 选择被 {} 包裹的区域(含花括号)
-   ib - 选择被 () 包裹的区域(不含括号)
-   iB - 选择被 {} 包裹的区域(不含花括号)
-   Esc - 退出可视化模式

## 可视化模式命令

-   \> - 向右缩进
-   < - 向左缩进
-   y - 复制
-   d - 剪切
-   ~ - 大小写切换

## 寄存器

-   :reg - 显示寄存器内容
-   "xy - 复制内容到寄存器 x
-   "xp - 粘贴寄存器 x 中的内容

Tip 寄存器被存储在 ~/.viminfo 中, 在下次重启vim时仍会加载Tip 寄存器 0 存储上一次复制的值

## 标记

-   :marks - 标记列表
-   ma - 设置当前位置为标记 a
-   \`a - 跳转到标记 a 的位置
-   y\`a - 复制当前位置到标记 a 的内容

## 宏

-   qa - 录制宏 a
-   q - 停止录制宏
-   @a - 执行宏 a
-   @@ - 重新执行上次执行的宏

## 剪切, 复制, 粘贴

-   yy - 复制当前行
-   2yy - 复制 2 行
-   yw - 复制当前单词
-   y$ - 复制, 从光标位置到行末
-   p - 在光标后粘贴
-   P - 在光标前粘贴
-   dd - 剪切当前行
-   2dd - 剪切 2 行
-   dw - 剪切当前单词
-   D - 剪切, 从光标位置到行末
-   d$ - 剪切, 从光标位置到行末 (同D)
-   x - 剪切当前字符

## 退出

-   :w - 保存
-   :w !sudo tee % - 使用 sudo 保存当前文件
-   :wq or :x or ZZ - 保存并退出
-   :q - 退出(修改未保存时警告)
-   :q! or ZQ - 不保存强制退出
-   :wqa - write (save) and quit on all tabs

## 查找/替换

-   /pattern - 查找pattern
-   ?pattern - 向上查找pattern
-   \\vpattern - pattern 中的非字母数字字符被视为正则表达式特殊字符 (不需转义字符)
-   n - 查找下一个
-   N - 查找上一个
-   :%s/old/new/g - 替换全部
-   :%s/old/new/gc - (逐个)替换
-   :noh - 移除搜索结果的高亮显示

## 多文件搜索

-   :vimgrep /pattern/ {file} - 在多个文件中搜索 pattern

e.g. :vimgrep /foo/ \*\*/\*

-   :cn - 移动至下一个
-   :cp - 移动至上一个
-   :copen - 打开搜索结果列表

## 多文件

-   :e file - 新建缓冲区打开 filename
-   :bnext or :bn - 切换到下个缓冲区
-   :bprev or :bp - 切换到上个缓冲区
-   :bd - 关闭缓冲区
-   :ls - 列出所有打开的缓冲区
-   :sp file - 新建缓冲区打开 filename 并水平分割窗口
-   :vsp file - 新缓冲区打开 filename 并垂直分割窗口
-   Ctrl + ws - 水平分割窗口
-   Ctrl + ww - 在窗口间切换
-   Ctrl + wq - 关闭窗口
-   Ctrl + wv - 垂直分割窗口
-   Ctrl + wh - 切换到右侧窗口
-   Ctrl + wl - 切换到左侧窗口
-   Ctrl + wj - 切换到下侧窗口
-   Ctrl + wk - 切换到上侧窗口

