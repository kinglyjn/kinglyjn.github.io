# Mac下升级Python2.7到3.6

> Mac系统自带python2.7，本文目的是将自带的[Python](http://lib.csdn.net/base/python)升级到3.6版本。网上有本多的做法是让python2.7和python3.X两个版本共存，博主并不知道，是两版本共存好，还是直接升 级好，所以读者要慎重选择方法。



### 关闭Rootless机制

由于Mac下的python2.7 默认是安装在／System目录下的。但是Mac有个Rootless机制，默认不允许直接在／System下作修改。所以要先关闭Rootless机制。

关闭Rootless机制的方法: 

```shell
1）.重启电脑, 重启过程中按住command+R, 进入恢复模式 
2）.打开terminal，键入: csrutil disable 
3）.重启电脑
```

如果之后要再开启Rootless机制:

```shell
1）.重启电脑, 重启过程中按住command+R, 进入恢复模式 
2）.打开terminal，键入: csrutil enable 
3）.重启电脑
```

<br>



### 升级Python2.7到3.6

**下载安装python3.6**

从官网[https://www.python.org/downloads/](https://www.python.org/downloads/) 
下载pkg版本，并安装。安装选默认路径，会安装到/Library/Frameworks/[python](http://lib.csdn.net/base/python).framework/Versions/目录下

<br>



**删除python2.7**

```shell
sudo rm -R /System/Library/Frameworks/Python.framework/Versions/2.7
```

<br>



**移动python3.6**

```shell
sudo mv /Library/Frameworks/Python.framework/Versions/3.6 /System/Library/Frameworks/Python.framework/Versions
```

<br>



**修改文件所属的group**

设置Group为wheel，原来系统自带的就是这样的。

```shell
sudo chown -R root:wheel /System/Library/Frameworks/Python.framework/Versions/3.6
```

<br>



**更新一下current的link**

在Versions的目录里有一个Current的link，是指向当前的Python版本，原始是指向系统自带的Python2.7，我们把它删除后，link就失效了，所以需要重新链一下

```shell
sudo rm /System/Library/Frameworks/Python.framework/Versions/Current
sudo ln -s /System/Library/Frameworks/Python.framework/Versions/3.6 /System/Library/Frameworks/Python.framework/Versions/Current
```

<br>



**重新链接可执行文件**

1) 先把系统原来的执行文件删掉

```shell
sudo rm /usr/bin/pydoc
sudo rm /usr/bin/python
sudo rm /usr/bin/pythonw
sudo rm /usr/bin/python-config
```

2) 建立新的链接

```shell
sudo ln -s /System/Library/Frameworks/Python.framework/Versions/3.6/bin/pydoc3.6 /usr/bin/pydoc
sudo ln -s /System/Library/Frameworks/Python.framework/Versions/3.6/bin/python3.6 /usr/bin/python
sudo ln -s /System/Library/Frameworks/Python.framework/Versions/3.6/bin/pythonw3.6 /usr/bin/pythonw
sudo ln -s /System/Library/Frameworks/Python.framework/Versions/3.6/bin/python3.6m-config /usr/bin/python-config
```

<br>



**更新.bash_profile文件**

默认的bash_profile里python的bin是指向/Library/Frameworks/Python.framework/Versions/3.6/bin的。要改到/System/目录下

```shell
vim ~/.bash_profile (只要能编辑就行)插入新的Python路径

# Setting PATH for Python 3.6

# The orginal version is saved in .bash_profile.pysave
PATH="/System/Library/Frameworks/Python.framework/Versions/3.6/bin:${PATH}"
export PATH
```

我默认是没有.bash_profile这个文件的，直接自己创建。

<br>



### 卸载pkg安装的python3.6

这一步不做，在使用pip3命令时还是要出错的(它默认连接到/Library/目录下照pip3命令，但是实际上应该到/System/Library/目录下找)。博主掉这个坑好久。我用的是CleanApp这个软件来卸载原来pkg安装的python3.6，安装进来的两个软件都卸载。

<br>



### Mac OSX python多版本管理工具：pyenv 和 virtualenv搭建

#### Installation pyenv

方法1：使用Mac OSX的Homebrew安装

```shell
$ brew update
$ brew install pyenv
```

方法2：通过github工程安装

```shell
# 下载
$ git clone https://github.com/yyuu/pyenv.git ~/.pyenv

# 设置环境变量
$ echo 'export PYENV_ROOT="$HOME/.pyenv"' >> ~/.bash_profile
$ echo 'export PATH="$PYENV_ROOT/bin:$PATH"' >> ~/.bash_profile
$ echo 'eval "$(pyenv init -)"' >> ~/.bash_profile

# restart your shell so the path changes take effect
$ $SHELL -l 或 $ exec $SHELL 
```

注：在编译 Python 过程中会依赖一些其他库文件，因而需要首先安装这些库文件，已知的一些需要预先安装的库如下。<br>

在 CentOS/RHEL/Fedora 下:

```shell
sudo yum install readline readline-devel readline-static
sudo yum install openssl openssl-devel openssl-static
sudo yum install sqlite-devel
sudo yum install bzip2-devel bzip2-libs
```

在 Ubuntu下：

```shell
sudo apt-get update
sudo apt-get install make build-essential libssl-dev zlib1g-dev
sudo apt-get install libbz2-dev libreadline-dev libsqlite3-dev wget curl
sudo apt-get install llvm libncurses5-dev libncursesw5-dev
```



#### 使用pyenv进行python版本管理

```shell
# 更新 pyenv 及其插件
$ pyenv update

# 查看有哪些版本可供安装
$ pyenv install --list

# 安装
$ pyenv install 3.6.2

# 在安装 Python 或者其他带有可执行文件的模块之后，需要对数据库进行更新才能生效	
$ pyenv rehash

# 卸载
$ pyenv uninstall 3.6.2

# 查看所有版本( * 表示当前使用的版本）
$ pyenv versions

# 切换python版本 
$ pyenv global 3.6.2 
$ pyenv global system
$ pyenv local 3.6.2
$ pyenv shell 3.6.2
```



#### 使用 python

- 输入 `python` 即可使用新版本的 python；
- 系统自带的脚本会以 `/usr/bin/python` 的方式直接调用老版本的 python，因而不会对系统脚本产生影响；
- 使用 `pip` 安装第三方模块时会自动按照到当前的python版本下，不会和系统模块发生冲突。
- 使用 `pip` 安装模块后，可能需要执行 `pyenv rehash` 更新数据库

