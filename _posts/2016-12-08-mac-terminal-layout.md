---
layout: post
title:  "mac terminal的样式设置"
desc: "mac terminal的样式设置"
keywords: "terminal layout,mac"
date: 2016-12-08
categories: [Other]
tags: [Other]
icon: fa-apple
---

### 第一步：偏好设置

最基本的，进入终端－偏好设置－描述文件－文本，在这儿可以设置Terminal的背景颜色，基本字体颜色，透明度等等，这些比较简单，不做过多介绍，看图<br>
![mac_terminal_layout_01](http://ww3.sinaimg.cn/mw690/e6de3f25jw1fakdricj9qj20m80jpjuf.jpg)

### 第二步：ls颜色
更改ls显示目录时的颜色

```shell
vim ~/.bash_profile

#在文件末尾添加以下三行代码
export LS_OPTIONS='--color=auto' # 如果没有指定，则自动选择颜色
export CLICOLOR='Yes' #是否输出颜色:
export LSCOLORS='exfxcxdxbxegedabagacad' #指定颜色

#按esc键退出修改模式，输入:wq!保存退出，输入下面代码使配置生效
source .bash_profile
```

详细说下第3小步LSCOLORS中的值代表的意思，这22个字母2个字母一组分别指定一种类型的文件或者文件夹显示的字体颜色和背景颜色。
从第1组到第11组分别指定的文件或文件类型为：<br>
    directory<br>
    symbolic link<br>
    socket<br>
    pipe<br>
    executable<br>
    block special<br>
    character special<br>
    executable with setuid bit set<br>
    executable with setgid bit set<br>
    directory writable to others, with sticky bit<br>
    directory writable to others, without sticky bit<br>

下面是颜色的子母对照：<br>
    a 黑色<br>
    b 红色<br>
    c 绿色<br>
    d 棕色<br>
    e 蓝色<br>
    f 洋红色<br>
    g 青色<br>
    h 浅灰色<br>
    A 黑色粗体<br>
    B 红色粗体<br>
    C 绿色粗体<br>
    D 棕色粗体<br>
    E 蓝色粗体<br>
    F 洋红色粗体<br>
    G 青色粗体<br>
    H 浅灰色粗体<br>
    x 系统默认颜色<br>
    所以，如果我们想把目录显示成红色，就可以把LSCOLORS设置为bxfxaxdxcxegedabagacad就可以了<br>


### 第三步： 设置vim颜色等等。即vim指令编辑文件的模式的显示效果

1.还是一样，进入用户主目录 cd ~
2.执行以下指令复制系统的vim配置文件到用户目录。
3.编辑.vimrc文件配置我们的vim。i键进入修改模式，在文件末尾添加上相应的vim配置。添加完成之后照样是esc之后:wq!保存退出再次vim就能看到效果

```shell
cp -r /usr/share/vim/vimrc ~/.vimrc
vim .vimrc
```

vim的配置选项特别多，我整理了一小部分给大家，代码如下，选取自己需要的，注释都写的很清楚，其中colorscheme参数比较看到效果，这个是vim的主题，vim的主题在/usr/share/vim/vim73/colors目录，所以这些名字的主题都是可以直接用的，先放个我用的pablo主题的效果图
<br>

![mac_terminal_layout_02](http://ww3.sinaimg.cn/mw690/e6de3f25jw1fake3q0miej20m80exadp.jpg)

