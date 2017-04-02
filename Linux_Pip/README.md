### Linux安装Python包管理工具——Pip

pip 是一个常用的Python包管理工具，主要是用于安装 PyPI 上的软件包，可以替代 easy_install 工具。

* GitHub: https://github.com/pypa/pip

* Doc: https://pip.pypa.io/en/latest/

在Centos 7中安装Python包管理工具—Pip时，使用 sudo yum install python-pip 时无法安装，是由于Centos发行版的源内容更新的比较滞后，导致找不到 python-pip 这个包管理工具。

这里解决的是无法安装pip 时问题，或者说是如何在Centos 7中安装Python包管理工具Pip 。

#### 一、脚本安装pip

首先下载get-pip.py脚本，然后运行即可安装：

	$  curl -O https://bootstrap.pypa.io/get-pip.py
	
	$  python get-pip.py

### 二、使用包管理软件安装

1、首先安装epel扩展源：

	$ sudo yum -y install epel-release


2、然后安装python-pip

	$ sudo yum -y install python-pip

#### 三、其他linux中安装 Pip
1、Debian/Ubuntu

	sudo apt-get install python-pip

2、Fedora

* Fedora 21:

	Python 2:

		sudo yum upgrade python-setuptools
		sudo yum install python-pip python-wheel
	
	Python 3:

		sudo yum install python3 python3-wheel

* Fedora 22:

	Python 2:

		sudo dnf upgrade python-setuptools
		sudo dnf install python-pip python-wheel

	Python 3:

		sudo dnf install python3 python3-wheel

3、openSUSE

	Python 2:

		sudo zypper install python-pip python-setuptools python-wheel
	
	Python 3:

		sudo zypper install python3-pip python3-setuptools python3-wheel


4、Arch Linux

	Python 2:

		sudo pacman -S python2-pip
	
	Python 3:

		sudo pacman -S python-pip
 

参考文档：

Installing pip/setuptools/wheel with Linux Package Managers