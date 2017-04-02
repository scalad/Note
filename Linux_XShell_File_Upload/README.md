### Xshell实现Windows上传文件到Linux主机
经常有这样的需求，我们在Windows下载的软件包，如何上传到远程Linux主机上？还有如何从Linux主机下载软件包到Windows下；之前我的做法现在看来好笨好繁琐，不过也达到了目的，笨人有本方法嘛；

我是怎么操作的：

1、打开一台本地Linux虚拟机，使用mount 挂载Windows的共享文件夹到Linux上，然后拷贝数据到Linux虚拟机里面；（经常第一步都不顺利，无法挂载Windows的文件夹）

2、在本地Linux虚拟机使用rsync同步拷贝的数据到远程Linux主机上，需要双方都要安装rsync包、openssh-clients包；遇到大一点的文件拷贝很费时间；

3、还有一种方法就是直接使用wget直接下载，提前是有下载的网址；大部分都是下载到Windows本地然后上传到远程Linux主机；

下面介绍一个简单的方法，方便上传Windows的文件到Linux上，也可以从Linux下载到Windows本地；

1、使用我们常用的Xshell登录工具，新建立一个远程会话，填写ip地址及用户名密码后，选择最下面的ZMODEM，填写下载的路径，加载的路径；2个路径可以一样也可以不一样；

![](https://github.com/silence940109/Java/blob/master/Linux_XShell_File_Upload/image/1.jpg)

2、在Linux主机上，安装上传下载工具包rz及sz

如果不知道你要安装包的具体名称，可以使用yum provides */name 进行查找系统自带软件包的信息；

[root@localhost src]# yum provides */rz

lrzsz-0.12.20-27.1.el6.i686 : The lrz and lsz modem communications programs

Repo        : base

Filename    : /usr/bin/rz

一般会列出软件包的名称及版本，还有安装路径；查询到软件包名后，使用yum install -y 包名 进行安装。

lrzsz包安装完成后包括上传rz、下载sz命令；只需要安装这个包即可。

[root@localhost src]# yum install -y lrzsz

3、从Windows上传文件，上传命令为rz；在Linux命令行下输入rz，上传的文件在当前命令行的目录下；

[root@localhost src]# rz

输入rz命令后，会弹出对话框，选择你要上传的文件，选择打开就上传到Linux主机。上传完可以使用ls 查看；

![](https://github.com/silence940109/Java/blob/master/Linux_XShell_File_Upload/image/2.jpg)

4、从Linux主机下载文件，下载命令为sz ，后面跟要下载的文件名；可以选择下载的保存文件夹；

[root@localhost src]# sz nginx-1.6.2.tar.gz

![](https://github.com/silence940109/Java/blob/master/Linux_XShell_File_Upload/image/3.jpg)
