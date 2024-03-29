---
layout: post
permalink: /vim-basic-commands/
title: Vim 基本命令
date: 2020-07-18T01:47:00
lang: zh-CN
---

注意，Vim 区分大小写。

## 移动

### 方向键移动

- `h` 或 `←` 光标左移
- `l` 或 `→` 光标右移
- `j` 或 `↓` 光标下移
- `k` 或 `↑` 光标上移

### 单词移动

- `w` (“word”) 光标向右移动一个单词
- `b` (“back”) 光标向左移动一个单词
- `e` (“end”) 移动光标到当前单词的最后一个字母

### 行首行末移动
类似正则表达式

- `^` 移动光标到行首
- `$` 移动光标到行末

### 屏幕位置移动

- `H` (“high”) 移动光标到屏幕上端
- `M` (“middle”) 移动光标到屏幕中端
- `L` (“low”) 移动光标到屏幕下端

### 页面滚动

- `Ctrl-f` (“forward”)  向下翻页(整个屏幕)
- `Ctrl-d` (“down”)  向下翻半页(半个屏幕)
- `Ctrl-b` (“backward”)  向上翻页(整个屏幕)
- `Ctrl-u` (“up”)  向上翻半页(半个屏幕)

## 插入文本

- `a` 在光标右侧插入文本
- `A` 在行末插入文本
- `i` 在光标左侧插入文本
- `I` 在行首插入文本
- `o` 在光标下插入新行
- `O` 在光标上插入新行

## 修改文本

- `cw` 删除当前单词的光标右侧部分，进入编辑模式
- `cc` 将当前行替换为空行，进入编辑模式
- `s` 删除当前字母，进入编辑模式
- `r` 替换当前字母，输入一个字母后自动返回命令模式

## 撤销修改

- `u` 撤销上次修改
- `U` 撤销对当前行的所有修改
- `Ctrl-r` 恢复上次修改

## 删除文本

### 删除字母

- `x` 删除光标右侧字母
- `X` 删除光标左侧字母

### 删除单词

- `dw` ("delete word") 删除当前单词的光标右侧部分 (`cw` 会进入编辑模式)
- `daw` ("delete a word") 删除光标所在的整个单词 (包括该单词后面的空格)
- `diw` ("delete inside word") 删除光标所在的整个单词 (不包括该单词后面的空格)

### 删除行

- `dd` 删除一行
- `dt<char>` 删除当前行光标到指定字母 `<char>`

## 参考
- https://docs.oracle.com/cd/E19683-01/806-7612/editorvi-43/index.html
- https://til.hashrocket.com/posts/fbfwnjxgtd-deleting-words-in-vim

