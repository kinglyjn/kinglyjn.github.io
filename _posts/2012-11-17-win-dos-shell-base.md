---
layout: post
title:  "常用dos命令"
desc: "常用dos命令"
keywords: "常用dos命令,dos,win,kinglyjn"
date: 2012-11-17
categories: [Linux]
tags: [win]
icon: fa-windows
---

```default
dos不显示中文：
运行regedit->HKEY_CURRENT_USER->Console->SystemRoot%_system32_cmd.exe->CodePage->修改为16进制的3a8

help //显示命令提示帮助
help xcopy //显示对xcopy命令的提示帮助
tree //显示当前文件夹树结构
tree /f abc //显示相对路径abc文件夹的树结构（连同显示文件）

start "" "刘德华 - 来生缘.mp3" //打开文件(含有空格的文件或文件夹路径)
start e:\Test\1  //打开1文件夹

dir /s //显示当前文件夹及其子文件夹下所有内容
		dir *.txt //找出当前路径所有txt文件
		dir *txt* //找出包含txt字符串的文件
		dir ??.txt //找出含有两个字符的txt文件
		//【注】通配符? * 也适用于下面的命令

md abc //创建abc文件夹
md abc\efg //创建多层文件夹

rd abc //删除文件夹 //注意文件夹中不能有内容
rd abc\def //删除abc下的def文件夹
rd /s e:\Test\1\2 //删除1文件夹下的2文件夹(2文件夹及其里面的所有内容被删掉) //有待确认
rd /s/q e:\Test\1\2 //删除1文件夹下的2文件夹(2文件夹及其里面的所有内容被删掉) //安静模式删除

md a..\\  //新建一个顽固文件夹
rd /s a..\\ //删除顽固文件夹
		
		
del abc.txt //删除abc文件
del abc //删除abc文件夹及其子文件夹下所有的（文件）内容（不包括文件夹）
del *.* //删除当前目录下的所有文件

edit abc.txt //创建编辑一个文本文件
edit e:\Test\Test.java //编辑Test.java文件
Alt //激活菜单选项

xcopy e:\Test\hello.txt d: //复制e:盘文件hello.txt到d:盘（仅仅复制文件夹下的文件）
xcopy /s e:\Test\3 e:\Test\1 //复制3文件夹中的全部(非空)内容(不包括3文件夹本身)到指定文件夹
xcopy /e e:\Test\3 e:\Test\1 //复制3文件夹中的全部(包括空文件夹)内容(不包括3文件夹本身)到指定文件夹


move 2\hello.txt e:\Test\1 //移动(剪切)文件
move e:\Test\2 e:\Test\1 //移动(剪切)文件夹(连同文件一起被移动)

expand 1\哈哈.rar 1\poem.txt //解压工具

rename 1\ping.dat ping.txt //文件(文件夹)重命名


/**
* 环境变量
*/
当我们执行一个文件时，电脑先在当前目录下查找这个文件，找到则执行，如果没有找到，则电脑按照path命令所指定的目录顺序去查找，先在C盘dos目录下，然后在windows目录下，最后在C盘根目录下寻找这个文件。但是，每次输入path路径仍是件很麻烦的事情，记得我们第一课讲过的DOS启动顺序中要自动执行的一个命令文件吗？对了，就是autoexec.bat,我们把path命令写在该文件里，这样启动计算机后，你就可以执行Path
命令指定目录中的任何可执行文件了：
//仅对当前窗口有效，若想永久生效就需要设置环境变量
path=c:\dos;c:\windows;c:\

cls //清屏
|more//分屏显示，在命令后面加

attrib +h *.* /s     ///将当前路径下的文件及其子文件夹下的文件设为隐藏的
attrib +h *.*	      //将当前路径下的文件设为隐藏的
		
attrib +r或-r [文件名]  //设置文件属性是否为只读
attrib +s或-s [文件名]  //设置文件属性是否为系统文件
attrib +a或-a [文件名]  //设置文件属性是否为归档文件
attrib /s  		//设置包括子目录的文件在内的文件属性 

date //显示修改当前日期
time //显示修改当前时间

ipconfig //查看ip信息


【chkdsk e: 】//检查你的磁盘使用情况

【mem】 //查看磁盘空间

【format c:】 //格式化c磁盘

【defrag】　 磁盘碎片整理 
［适用场合］　磁盘读写次数很多，或磁盘使用时间很长了，可能需要使用这条命令
　　　　　　　整理磁盘。磁盘碎片并不是指磁盘坏了，而只是由于多次的拷贝和删
　　　　　　　除文件后，磁盘使用会很不连贯，致使速度变慢。 
［用　　法］　1. C:\>defrag 
　　　　　　　2. 选择要整理的磁盘 
			3. 电脑分析磁盘状况，然后告诉我们磁盘有多少需整理。按Esc键
		    4. 选择Optimization Method(磁盘优化方法)，选择“全部优化”
　　　　　　　　 或“仅优化文件”
		    5. 选择Begin Optimization 开始整理 
　　　　　　　6. 整理完后，按回车键 

【doskey】　 调用和建立DOS宏命令 
［适用场合］　　经常需要输入重复的命令时，有非常大的用处 
［用　　法］　　doskey　 
　　　　　　　　将doskey驻留内存，开辟出缓冲区，以后输入的命令都将保存在缓冲
　　　　　　　　区中，可以随时调用 
　　　　　　　　doskey [宏命令名]=[命令名] 　 
　　　　　　　　将宏命令定义为命令，以后输入宏命令，电脑就会执行相应的命令 
　　　　　　　　doskey /reinstall　　　　　　重新安装doskey 
　　　　　　　　doskey /bufsize= 　　　　　　设置缓冲区的大小 
　　　　　　　　doskey /macros 　　　　　　　显示所有doskey宏 
　　　　　　　　doskey /history 　　　 　　　显示内存中所有命令 
　　　　　　　　doskey /insert|overstrike 设置新键入的字符是否覆盖旧的字符 
［例　　子］　　C:\>DOSKEY 
　　　　　　　　C:\>dir 
　　　　　　　　C:\>copy C:\temp\*.* a: 
　　　　　　　　C:\>del c:\temp\*.* 
　　　　　　　　C:\>copy b:\*.* c:\temp 
　　　　　　　　上述四条命令都已被保存，用光标控制键的上下可以依次选择使用或
　　　　　　　　修改, 也可以用F7键列出保存的所有命令
　　　　　　　　C:\>doskey di=dir/w/p 定义di为宏命令，意思是执行dir/w/p 
		
		
【fdisk】 　硬盘分区 
［建　　议］　　只有硬盘被很利害的病毒感染时，或是一块新硬盘才需要分区，最好
　　　　　　　　请懂行的人指导。硬盘都需经过低级格式化，分区，格式化三个步骤
　　　　　　　　才可使用，成品电脑内的硬盘都已经做过这些加工了。 
［用　　法］　　输入fdisk后按回车即可进入提示界面


【prompt】　设置提示符 
［适用场合］　　当你厌烦了c:\>的提示符或者您想使您的提示符与众不同时，您可以
　　　　　　　　试一试，非常有趣的DOS命令，可以随时显示时间与日期。 
［用　　法］　　prompt $p$g 以当前目录名和>号为提示符，这是最常用的提示符 
　　　　　　　　prompt $t 表示时间　　　　　　prompt $d 表示日期 
　　　　　　　　prompt $$ 表示$ 　　　　　　　prompt $q 表示= 
　　　　　　　　prompt $v 表示当前版本　　　　prompt $l 表示< 
　　　　　　　　prompt $b 表示| 　　　　　　　prompt $h 表示退位符 
　　　　　　　　prompt $e 表示Esc代表的字符 　prompt $_ 表示回车换行 
［例　　子］　　C:\DOS>prompt wang$g 将wang>作为提示符 
　　　　　　　　WANG>prompt $t$d$g 　使用时间、日期和>号做为提示符 
　　　　　　　　0:01:07.77Thu 08-29-1996>prompt $p$g 
　　　　　　　　C:\DOS> 

【debug】　程序调试命令 
		［建　　议］　　如果你学过汇编语言，那你应该会使用debug，如果没学过，最好别
		　　　　　　　　使用 
		［用　　法］　　debug [文件名]



使用批处理文件bat/cmd
	====================================================================================
	echo		//表示显示此命令后的字符
	echo off	//表示在此语句后所有运行的命令都不显示命令行本身
	@			//与echo off相象，但它是加在其它命令行的最前面，表示运行时不显示命令行本身。
	call		//调用另一条批处理文件（如果直接调用别的批处理文件 ，执行完那条文件后将无法执行当前文件后续命令）
	pause		//运行此句会暂停，显示Press any key to continue... 等待用户按任意键后继续 
	rem			//表示此命令后的字符为解释行，不执行，只是给自己今后查找用的 
	====================================================================================

例如：
	echo off				   //不显示命令行
	dir c:\*.* >e:\Test\a.txt  //将c:盘的文件列表写入e:盘下的a.txt
	call e:\Test\start.bat	   //调用e:盘下的start.bat文件
	echo 你好				   //显示你好
	pause					   //暂停,等待按键继续
	rem  这是注释的内容		   //注释,在dos下不显示,只是给编程者提供方便
	cd e:\Test\1			   //进入指定目录

   批处理文件中还可以像C语言一样使用参数，这只需用到一个参数表示符%。 
　　%表示参数，参数是指在运行批处理文件时在文件名后加的字符串。
	变量可以从 %0到%9，%0表示文件名本身，字符串用%1到%9顺序表示。 
　　	例如，C：根目录下一批处理文件名为f.bat，内容为 format %1 
　　	则如果执行C:\>f a: 　　　则实际执行的是format a: 
　　	又如C：根目录下一批处理文件的名为t.bat，内容为 type %1 type %2 
　　	那么运行C:\>t a.txt b.txt 将顺序地显示a.txt和b.txt文件的内容 


//if goto choice for 是批处理文件中比较高级的命令，如果这几个你用得很熟练，你就是批处理文件的专家啦。 
If/If not---表示将判断是否符合规定的条件，从而决定执行不同的命令。 有三种格式: 
1、if "参数" == "字符串" 　待执行的命令 
   参数如果等于指定的字符串，则条件成立，运行命令，否则运行下一句。(注意是两个等号）
   如if "%1"=="a" format a: 
2、if exist 文件名　 待执行的命令 
   如果有指定的文件，则条件成立，运行命令，否则运行下一句。如if exist config.sys edit config.sys 
3、if errorlevel 数字　 待执行的命令 
	如果返回码等于指定的数字，则条件成立，运行命令，否则运行下一句。如if errorlevel 2 goto x2 　
	DOS程序运行时都会返回一个数字给DOS，称为错误码errorlevel或称返回码

goto--------批处理文件运行到这里将跳到goto 所指定的标号处， 一般与if配合使用。 如:
			goto end 
			:end 
			echo this is the end
			标号用 :字符串 表示，标号所在行不被执行

choice------使用此命令可以让用户输入一个字符，从而运行不同的命令。使用时应该加/c:参数，
			c:后应写提示可输入的字符，之间无空格。它的返回码为1234……
			如: choice /c:dme defrag,mem,end
			将显示
			defrag,mem,end[D,M,E]?
			例如，test.bat的内容如下: 
			@echo off 
			choice /c:dme defrag,mem,end 
			if errorlevel 3 goto defrag 应先判断数值最高的错误码
			if errorlevel 2 goto mem 
			if errotlevel 1 goto end 
			:defrag 
			c:\dos\defrag 
			goto end 
			:mem 
			mem 
			goto end 
			:end 
			echo good bye
			此文件运行后，将显示 defrag,mem,end[D,M,E]? 用户可选择d m e ，然后if语句将作出判断，d表示执行标号为defrag的程序段，m表示执行标号为mem的程序段，e表示执行标号为end的程序段，每个程序段最后都以goto end将程序跳到end标号处，然后程序将显示good bye，文件结束。

for--------循环命令，只要条件符合，它将多次执行同一命令。 
			   格式FOR [%%f] in (集合) DO [命令] 
			   只要参数f在指定的集合内，则条件成立，执行命令 
			   如果一条批处理文件中有一行: 
			   for %%c in (*.bat *.txt) do type %%c 
			   含义是如果是以bat或txt结尾的文件，则显示文件的内容。
```
<br>