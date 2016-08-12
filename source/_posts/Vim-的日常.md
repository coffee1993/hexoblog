title: 我的Vim之路
date: 2016-03-15 13:03:01
tags:
categories:
- tools
---

## 1 为何选择VIM

选择VIM这条道路,是公司环境和一个程序员的自我修养之必然，磨炼其意志，考验其记忆的一项专业技能，我立志要成为一名Vimer

就为了我花重金买的Flico蓝牙键盘和大屏幕，我也要坚持下去。


我很信奉这些话：

- 使用Vim最重要的一点是别碰上下左右箭头键
- 捣鼓下vimrc，弄下插件， 就会变得很优雅。 之后再慢慢地，你就会发现，这些优雅并不实用，最后还是会回归根源。最后蓦然回首，你就会发现 原本的vim，才是你所最求的优雅。而你也成为了一名优秀的vimer
- 实践起来，永远比记忆来得高效。

>来自知乎

<!-- more -->

## 2 Vim使用手册

- 感谢前辈们的付出，踩在巨人的肩膀上。
- [转 vim使用手册](http://www.ldc4.cn/2016/03/11/VimStart/)
  一张图看懂vim
  ![](http://7xrw2w.com1.z0.glb.clouddn.com/Vim_QQ%E5%9B%BE%E7%89%8720160406201932.jpg)

  一张图不够? 再来一张

  ![](http://7xrw2w.com1.z0.glb.clouddn.com/Vim_vi_tutorial.png)

## 3 Vim 环境配置记录


### ~/.viminfo

vim会将曾经做过的行为记录下来，记录操作放在~/.viminfo

## 4 配置文件

### ~/.vimrc
```
"一般设置"

set nu  "行号
set backspace=2 "0 1 2 0和1仅可以删除刚才输入的字符 2表示可以删除任意字符"
set autoindent "自动缩排 noautoindent 非自动缩排"
set backup "是否自动保存备份文件 backup为保存，系统会在同一目录下产生filename~的文件
set hlsearch "将查找的字符串反白"



"状态设置"
set showmode "左下角哪一行的状态 INSERT  NORMAL等
set ruler "可显示右下角的状态栏


"颜色设置
set bg=dark "显示不同的底色色调 默认是light 可控制批注颜色 如果为dark 批注颜色就为天蓝色 默认为深蓝色 不容易看见"
syntax on "程序语法 颜色显示"


```


### 插件及插件管理 Vunble

- 在Home目录创建~/.vim目录和.vimrc文件

- Vunble作用：
  更新，搜索，移除，vim插件

- 创建目录环境
  ```
  git clone https://github.com/gmarik/vundle.git ~/.vim/bundle/vundle
  ```
- 在.vimrc配置文件中添加vundle支持
  ```
  filetype off
  set rtp+=~/.vim/bundle/vundle/
  call vundle#rc()
  ```
  也可以在.vimrc文件添加：

  ``  
  if filereadable(expand("~/.vimrc.bundles"))
  source ~/.vimrc.bundles
  endif
  ``
  然后创建 .vimrc.bundle 并添加：

  ```
  filetype off
  set rtp+=~/.vim/bundle/vundle/
  call vundle#rc()
  ```
  vim命令行输入：
  ```
  :BundleInstall
  ```
  完成安装

### 安装插件

  Bundle 分为三类：
  - 1.github vim-scripts repos ,只需要写出repos名称
  - 2.在github其他用户下的repos，需要写出用户名/repos名
  - 3.不在github上的插件，需要写出git全路径

### 安装插件记录

- 1.自动补全 Bundle 'Valloric/YouCompleteMe'
  - YouCompleteMe已经集成了Syntastic
- 2.括号匹配 Bundle 'Raimondi/delimitMate

- 3.格式化 Bundle 'Chiel92/vim-autoformat'
  Formatter 是一个格式化程序框架，依赖于代码检测工具，所以需要单独安装这些工具 astyle、js-beautify等，下面有github连接，有安装方法及列表
  [代码检测工具列表 https://github.com/Chiel92/vim-autoformat#default-formatprograms](https://github.com/Chiel92/vim-autoformat#default-formatprograms)

  - 其中 [astyle 官网下载](http://astyle.sourceforge.net/)并没有看到关于linux版本的下载，我从其他渠道下载后共享出来：
  - [astyle 百度网盘下载](https://pan.baidu.com/s/1jIaXh9G)
  - [astyle 微云网盘下载](http://url.cn/2J9Zmik)

  安装：
  - 解压 tar -zxvf astyle_2.05.1_linux.tar.gz
  - cd build 目录
  - make

  添加一键格式化 ~/.vimrc
  - nnoremap <F3> :Autoformat<CR>

- 4.语法检测 Bundle 'scrooloose/syntastic' "集成于YouCompleteMe"

- 5.当前目录下的树形目录 Bundle "scrooloose/nerdtree"

  - 设置F5开启或者关闭树形目录 nmap <F5> :NERDTreeToggle<CR>
  - 常用快捷键：
    - 通过hjkl来移动光标
    - o 打开关闭文件或目录，如果想打开文件，必须光标移动到文件名
    - t 在标签页中打开
    - s和i可以水平或纵向分割窗口打开文件
    - p到上层目录
    - P到根目录
    - K到同目录第一个节点
    - P到同目录最后一个节点
- 6.状态栏插件 Bundle "Lokaltog/vim-powerline"
- 7.Bundle 'danro/rename.vim' "重命令 :rename[!] {newname}



### Ctags 的介绍和使用

- 何为Ctags
  - 程序源代码树产生索引文件（或tag文件),在产生的tag文件中，每一个tag的入口指向了一个编程语言的对象。这个对象可以是变量定义、函数、类或其他的物件。
  - 简单的说就是：产生标记文件以帮助在源文件中定位对象
  - 作用就是查看函数，变量的定义位置。

  [Ctags 维基百科](https://zh.wikipedia.org/wiki/Ctags)

- 安装

  - ``sudo apt-get install exuberant-ctags``

- 使用

  - 熟练的使用ctags仅需记住下面七条命令：
       - ctags –R *    ($ 为Linux系统Shell提示符)
       - vi –t tag       (请把tag替换为您欲查找的变量或函数名)
       - ：ts                (ts 助记字：tags list, “:”开头的命令为VI中命令行模式命令)
       - ：tp                (tp 助记字：tags preview)---此命令不常用，可以不用记
       - ：tn                (tn 助记字：tags next) ---此命令不常用，可以不用记
       - Ctrl + ]    跟进
       - Ctrl + T    返回

 - 相关博客链接：
[在Vim中使用ctags](http://www.vimer.cn/2009/10/%E5%9C%A8vim%E4%B8%AD%E4%BD%BF%E7%94%A8ctags.html)
[标签：ctags](http://easwy.com/blog/archives/tag/ctags/)

### taglist

  - 使用taglist之前
    - 打开文件类型自动检测：filetype on
    - 已经安装过了Ctags,taglist需要它生产的tags文件
    - 你的vim支持system()调用

  - vimrc配置：
    ```
    Bundle 'taglist.vim'
    let Tlist_Ctags_Cmd='ctags'
    let Tlist_Show_One_File=1           	"不同时显示多个文件的tag，只显示当前文件的
    let Tlist_WinWidt =28					"设置taglist的宽度
    let Tlist_Exit_OnlyWindow=1         	"如果taglist窗口是最后一个窗口，则退出vim
    "let Tlist_Use_Right_Window=1			"在右侧窗口中显示taglist窗口
    let Tlist_Use_Left_Windo =1        		"在左侧窗口中显示taglist窗口
    ```

 - 相关命令操作：
    - 打开 taglist窗口 :TlistOpen
      ![](http://7xrw2w.com1.z0.glb.clouddn.com/Vim_taglist.jpg)
    - <CR>          跳到光标下tag所定义的位置，用鼠标双击此tag功能也一样
    - o             在一个新打开的窗口中显示光标下tag
    - <Space>       显示光标下tag的原型定义
    - u             更新taglist窗口中的tag
    - s             更改排序方式，在按名字排序和按出现顺序排序间切换
    - x             taglist窗口放大和缩小，方便查看较长的tag
    - +             打开一个折叠，同zo
    - -             将tag折叠起来，同zc
    - *             打开所有的折叠，同zR
    - =             将所有tag折叠起来，同zM
    - [[            跳到前一个文件
    - ]]            跳到后一个文件
    - q             关闭taglist窗口
    - <F1>          显示帮助


  - 相关博客链接
    [vi/vim使用进阶: 使用taglist插件](http://easwy.com/blog/archives/advanced-vim-skills-taglist-plugin/)


### Cscope 的介绍和使用

[Cscope官网](http://cscope.sourceforge.net/)

安装
sudo apt-get install cscope

使用
cd 源码目录
cscope -Rbq
附常用的命令：
：cs find s ---- 查找C语言符号，即查找函数名、宏、枚举值等出现的地方
：cs find g ---- 查找这个定义，及查找函数、宏、枚举等定义的位置，类似ctags所提供的功能
：cs find d ---- 查找本函数调用的函数们
：cs find c ---- 查找调用本函数的函数们
：cs find t ---- 查找指定的字符串
：cs find e ---- 查找egrep模式，相当于egrep功能，但查找速度快多了
：cs find f ---- 查找并打开文件，类似vim的find功能
：cs find i ---- 查找包含include这个文件的文件们

vim中使用cscope

cscope -b这个命令会生成cscope.out 数据库文件
vim中使用需要:cs add cscope.out

"Cscope配置"

```
"-- Cscope setting --
if has("cscope")
set csprg=/usr/bin/cscope " 指定用来执行cscope的命令
set csto=0 " 设置cstag命令查找次序：0先找cscope数据库再找标签文件；1先找标签文件再找cscope数据库
set cst " 同时搜索cscope数据库和标签文件
set cscopequickfix=s-,c-,d-,i-,t-,e- " 使用QuickFix窗口来显示cscope查找结果
set nocsverb
if filereadable("cscope.out") " 若当前目录下存在cscope数据库，添加该数据库到vim
cs add cscope.out
elseif $CSCOPE_DB != "" " 否则只要环境变量CSCOPE_DB不为空，则添加其指定的数据库到vim
cs add $CSCOPE_DB
endif
set csverb

endif

map <F4> :cs add ./cscope.out .<CR><CR><CR> :cs reset<CR>
imap <F4> <ESC>:cs add ./cscope.out .<CR><CR><CR> :cs reset<CR>

" 将:cs find c等Cscope查找命令映射为<C-_>c等快捷键（按法是先按Ctrl+Shift+-, 然后很快再按下c）
nmap <C-_>s :cs find s <C-R>=expand("<cword>")<CR><CR> :copen<CR><CR>
nmap <C-_>g :cs find g <C-R>=expand("<cword>")<CR><CR>
nmap <C-_>d :cs find d <C-R>=expand("<cword>")<CR><CR> :copen<CR><CR>
nmap <C-_>c :cs find c <C-R>=expand("<cword>")<CR><CR> :copen<CR><CR>
nmap <C-_>t :cs find t <C-R>=expand("<cword>")<CR><CR> :copen<CR><CR>
nmap <C-_>e :cs find e <C-R>=expand("<cword>")<CR><CR> :copen<CR><CR>
nmap <C-_>f :cs find f <C-R>=expand("<cfile>")<CR><CR>
nmap <C-_>i :cs find i <C-R>=expand("<cfile>")<CR><CR> :copen<CR><CR>
```
使用方式：

为源码建立一个cscope数据库
```
voidzhang@ubuntu:~/Android$ cscope -Rbq
voidzhang@ubuntu:~/Android$ ls cscope.*
cscope.in.out  cscope.out  cscope.po.out
```

用vim打开某个源码文件，末行模式下，输入`:cs add cscope.out`（该命令已被我们映射为快捷键F4），添加cscope数据库到vim。因为我们已将vim配置为启动时，自动添加当前目录下的cscope数据库，所以你再添加该cscope数据库时，vim会提示“重复cscope数据库 未被加入
完成前两步后，现在就可以用`cs find c`等Cscope查找命令查找关键字了。我们已在.vimrc中将`cs find c`等Cscope查找命令映射为`<C-_>c`等快捷键（按法是先按`Ctrl+Shift+-`, 然后很快按下c）

### vim 分屏使用
`:split` 文件名 纵向(上下)打开文件
`:vsplit` 文件名 横向(左右)打开文件
`<C+w> +` 方向切换光标

### 我目前的 ~/.vimrc

```
if(has("win32") || has("win95") || has("win64") || has("win16")) "判定当前操作系统类型
    let g:iswindows=1
else
	let g:iswindows=0
endif

set nocompatible "be iMproved

filetype on

set nu  "行号
set backspace=2 "0 1 2 0和1仅可以删除刚才输入的字符 2表示可以删除任意字符"
set autoindent "自动缩排 noautoindent 非自动缩排"
set nobackup "是否自动保存备份文件 backup为保存，系统会在同一目录下产生filename~的文件
set hlsearch "将查找的字符串反白"
"set incsearch "在输入要搜索的文字时，vim会实时匹配
set tabstop=4 "让一个tab等于4个空格
set shiftwidth=4  

"状态设置"
set showmode "左下角哪一行的状态 INSERT  NORMAL等
set ruler "可显示右下角的状态栏

"颜色设置
syntax on
set bg=dark "显示不同的底色色调 默认是light 可控制批注颜色 如果为dark 批注颜色就为天蓝色 默认为深蓝色 不容易看见"

colorscheme solarized

set tabstop=4 "硬TAB
set softtabstop=4 "软TAB



set rtp+=~/.vim/bundle/vundle/
call vundle#rc()

Bundle 'gmarik/vundle'

"My Bundles here :

"Bundle 'Valloric/YouCompleteMe'
Bundle 'Raimondi/delimitMate'
Bundle 'Chiel92/vim-autoformat'
Bundle 'scrooloose/nerdtree'
Bundle "Lokaltog/vim-powerline"
set laststatus=2
Bundle 'danro/rename.vim'
Bundle 'kien/ctrlp.vim'
Bundle 'taglist.vim'
let Tlist_Ctags_Cmd='ctags'
let Tlist_Show_One_File=1               "不同时显示多个文件的tag，只显示当前文件的
let Tlist_WinWidt =28                    "设置taglist的宽度
let Tlist_Exit_OnlyWindow=1             "如果taglist窗口是最后一个窗口，则退出vim
let Tlist_Use_Right_Window=1            "在右侧窗口中显示taglist窗口
"let Tlist_Use_Left_Windo =1                "在左侧窗口中显示taglist窗口"

set tags=tags; ";号一定记得不要漏掉

map <silent> <F9> :TlistToggle<CR>
nnoremap <F3> :Autoformat<CR>
nnoremap <F5> :NERDTreeToggle<CR>

if has("cscope")
set csprg=/usr/bin/cscope " 指定用来执行cscope的命令
set csto=0 " 设置cstag命令查找次序：0先找cscope数据库再找标签文件；1先找标签文件再找cscope数据库
set cst " 同时搜索cscope数据库和标签文件
set cscopequickfix=s-,c-,d-,i-,t-,e- " 使用QuickFix窗口来显示cscope查找结果
set nocsverb
if filereadable("cscope.out") " 若当前目录下存在cscope数据库，添加该数据库到vim
cs add cscope.out
elseif $CSCOPE_DB != "" " 否则只要环境变量CSCOPE_DB不为空，则添加其指定的数据库到vim
cs add $CSCOPE_DB
endif
set csverb
endif
map <F4> :cs add ./cscope.out .<CR><CR><CR> :cs reset<CR>
imap <F4> <ESC>:cs add ./cscope.out .<CR><CR><CR> :cs reset<CR>
" 将:cs find c等Cscope查找命令映射为<C-_>c等快捷键（按法是先按Ctrl+Shift+-, 然后很快再按下c）
nmap <C-_>s :cs find s <C-R>=expand("<cword>")<CR><CR> :copen<CR><CR>
nmap <C-_>g :cs find g <C-R>=expand("<cword>")<CR><CR>
nmap <C-_>d :cs find d <C-R>=expand("<cword>")<CR><CR> :copen<CR><CR>
nmap <C-_>c :cs find c <C-R>=expand("<cword>")<CR><CR> :copen<CR><CR>
nmap <C-_>t :cs find t <C-R>=expand("<cword>")<CR><CR> :copen<CR><CR>
nmap <C-_>e :cs find e <C-R>=expand("<cword>")<CR><CR> :copen<CR><CR>
nmap <C-_>f :cs find f <C-R>=expand("<cfile>")<CR><CR>
nmap <C-_>i :cs find i <C-R>=expand("<cfile>")<CR><CR> :copen<CR><CR>
```
