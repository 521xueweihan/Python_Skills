# Python Tools

## 介绍
下面列举了我在 Python 用到的工具，用以提高编码效率。

### pip
pip：python包管理。安装：`sudo easy_install pip`

### 虚拟环境
virtualenvwrapper：python虚拟环境管理。如果pip安装过程中遇到six安装问题，则可尝试使用brew安装python，或者
`pip install --ignore-installed six`

debian 系统下面安装virtualenv
`sudo apt install virtualenv -y`

python2.7 初始化 virtualenv 环境
`virtualenv env` 执行完该命令将在当前目录创建名为env的环境目录

python3.x 初始化 virtualenv 环境
`virtualenv -p python3 env` 执行完该命令将在当前目录创建名为env的环境目录

使用 virtualenv 
`source env/bin/active` 

退出 virtualenv
`deactivate` 或退出虚拟终端即可

### ipython
ipython：交互式python解释器，带有自动补全等高级功能。安装：`sudo pip install ipython`

### httpie
httpie：通过命令行测试web服务，[使用方法](https://httpie.org/doc#examples)

### 测试工具
unittest、coverage：第一个用于单元测试，第二个用于检测代码的测试覆盖率。
