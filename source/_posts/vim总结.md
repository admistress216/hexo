---
title: vim总结
date: 2018-09-04 16:38:11
tags:
---
### 1. 树形目录插件NERDTree
<br>
#### 1.1 下载及安装
```php
wget http://www.vim.org/scripts/download_script.php?src_id=17123 -O nerdtree.zip 
unzip nerdtree.zip
 
mkdir -p ~/.vim/{plugin,doc}
 
cp plugin/NERD_tree.vim ~/.vim/plugin/
cp doc/NERD_tree.txt ~/.vim/doc/
```

#### 1.2 使用
安装好后,直接在vim中输入:NERDTree,即可看到目录结构.

#### 1.3 快捷方式配置
在~/.vimrc中添加一下内容:
```php
"按F3即可显示/隐藏目录
map <F3> :NERDTreeMirror<CR>
map <F3> :NERDTreeToggle<CR>

```
### 2. 设置选项
```php
help 'option': 搜索设置选项(option可为hls,is等)
set showmode/noshowmode: 设置是否显示模式
set hls/hlsearch(highlight search): 设置高亮
set is/incsearch(incurrent search): 实时显示
set ic/ignorecase: 忽略大小写
set ruler: 右下角显示光标的位置

```

### 3. 常用按键设置
```php
x/X:删除光标下/光标前的字符
i/a/A:在光标之前/之后/词尾插入
diw/daw/dw:(删除光标所在的字符,不包括/包括空白字符)/(删除从光标开始到下个单词词首的所有)
D:删除导航为的内容
I/A:在当前行首/行尾插入
w/e/b/ge:(下个单词词首/词尾)/(上个单词词首/词尾)
0/^:移动到行首/非空行首
2$:光标移动到下一行的行尾
[number]f/F+character:当前行向左/向右查找字符并移动到字符位置,可以通过;/,来向后/向前重复移动
H/M/L:移动到所见内容的(屏幕内)首部,中部,尾部
zb/zz/zt:光标所在的行移动到底部/中部/顶部
*/#: 光标所在单词取出并作为向前/向后查找对象



```
