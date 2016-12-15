---
layout: post
title:  "Markdown日常用法示例"
desc: "Markdown日常用法示例"
keywords: "markdown,first blog"
date: 2016-12-05
categories: [Other]
tags: [Oher]
icon: fa-bookmark-o
---

### 文字标题和内容层次：

```shell
# 第一层
## 第二层
### 第三层
#### 第四层
##### 第五层
###### 第六层
```

# 第一层

## 第二层

### 第三层

#### 第四层

##### 第五层

###### 第六层

<br>


### 突出显示示例：<br>

```
`这是突出显示的内容`
```

`这是突出显示的内容`
<br><br>


### 普通代码高亮示例：

```
` ` `
[root@test ~]# rpm -ivh http://download.fedoraproject.org/pub/epel/6/i386/epel-release-6-5.noarch.rpm
[root@test ~]# rpm -ivh http://download.fedoraproject.org/pub/epel/6/i386/epel-release-6-8.noarch.rpm
` ` `
```

```
[root@test ~]# rpm -ivh http://download.fedoraproject.org/pub/epel/6/i386/epel-release-6-5.noarch.rpm
[root@test ~]# rpm -ivh http://download.fedoraproject.org/pub/epel/6/i386/epel-release-6-8.noarch.rpm
```
<br>


### java代码高亮示例：

```
` ` `java
public class Test {
	public static void main(String[] args) {
		System.out.println("Hello World!");
	}
}
` ` `
```

```java
public class Test {
	public static void main(String[] args) {
		System.out.println("Hello World!");
	}
}
```
<br>


### 图片示例：

```
![这里写图片描述](http://img.blog.csdn.net/20161212160702517?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2luZ2x5am4=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)图片后的文字图片后的文字图片后的文字图片后的文字图片后的文字图片后的文字图片后的文字图片后的文字图片后的文字图片后的文字图片后的文字
```

![这里写图片描述](http://img.blog.csdn.net/20161212160702517?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2luZ2x5am4=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)图片后的文字图片后的文字图片后的文字图片后的文字图片后的文字图片后的文字图片后的文字图片后的文字图片后的文字图片后的文字图片后的文字
<br><br>


### 链接示例：<br>

```
文字链接：[Kinglyjn Blog Home](https://kinglyjn.github.io)<br>
直接链接：<https://kinglyjn.github.io><br>
图片链接：<br> [![datactr alt](http://img.blog.csdn.net/20161212160910284?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2luZ2x5am4=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)](http://www.datactr.cn)
```

文字链接：[Kinglyjn Blog Home](https://kinglyjn.github.io)<br>
直接链接：<https://kinglyjn.github.io><br>
图片链接：<br> [![datactr alt](http://img.blog.csdn.net/20161212160910284?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2luZ2x5am4=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)](http://www.datactr.cn)
<br><br>


### 有序列表示例：

```
1. 第一项
2. 第二项
3. 第三项
```
1. 第一项
2. 第二项
3. 第三项


### 无序列表示例：

```
* AAAA
* BBBB
* CCCC
```
* AAAA
* BBBB
* CCCC

<br>

### 表格示例：

还是写html代码为好
<br><br>


### 引用示例：

```
> 朝辞白帝彩云间<br>
> 千里江陵一日还<br>
> 两岸猿声啼不住<br>
> 轻舟已过万重山<br>
```

> 朝辞白帝彩云间<br>
> 千里江陵一日还<br>
> 两岸猿声啼不住<br>
> 轻舟已过万重山<br>

<br>
<br>

附：markdown代码高亮支持的语言

```
Language		language_key
-----------------------------------------
1C					1c
ActionScript		actionscript
Apache				apache
AppleScript a		pplescript
AsciiDoc			asciidoc
AspectJ				asciidoc
AutoHotkey			autohotkey
AVR Assembler		avrasm
Axapta				axapta
Bash				bash
BrainFuck			brainfuck
Cap’n Proto			capnproto
Clojure REPL		clojure
Clojure				clojure
CMake				cmake
CoffeeScript		coffeescript
C++					cpp
C# c				s
CSS					css
D					d
Dart				d
Delphi				delphi
Diff				diff
Django				django
DOS .bat			dos
Dust				dust
Elixir				elixir
ERB (Embedded Ruby)	erb
Erlang REPL			erlang-repl
Erlang				erlang
FIX					fix
F#					fsharp
G-code (ISO 6983)	gcode
Gherkin				gherkin
GLSL				glsl
Go					go
Gradle				gradle
Groovy				groovy
Haml				haml
Handlebars			handlebars
Haskell				haskell
Haxe				haxe
HTTP				http
Ini file			ini
Java				java
JavaScript			javascript
JSON 				json
Lasso				lasso
Less				less
Lisp				lisp
LiveCode			livecodeserver
LiveScript			livescript
Lua					lua
Makefile			makefile
Markdown			markdown
Mathematica			mathematica
Matlab				matlab
MEL (Maya Embedded Language)	mel
Mercury				mercury
Mizar				mizar
Monkey				monkey
nginx				nginx
Nimrod				nimrod
Nix					nix
NSIS				nsis
Objective C			objectivec
OCaml				ocaml
Oxygene				oxygene
Parser 3			parser3
Perl				perl
PHP					php
PowerShell			powershell
Processing			processing
Python’s profiler output	profile
Protocol Buffers	protobuf
Puppet				puppet
Python				python
Q					q
R					r
RenderMan RIB		rib
Roboconf			roboconf
RenderMan RSL		rsl
Ruby				ruby
Oracle Rules Language	ruleslanguage
Rust				rust
Scala				scala
Scheme				scheme
Scilab				scilab
SCSS				scss
Smali				smali
SmallTalk			smalltalk
SML					sml
SQL					sql
Stata				stata
STEP Part 21 (ISO 10303-21)	step21
Stylus				stylus
Swift				swift
Tcl					tcl
TeX					tex
Thrift				thrift
Twig				twig
TypeScript			typescript
Vala				vala
VB.NET				vbnet
VBScript in HTML	vbscript-html
VBScript			vbscript
Verilog				verilog
VHDL				vhdl
Vim Script			vim
Intel x86 Assembly	x86asm
XL					xl
XML, HTML			xml
```


