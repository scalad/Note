### Linux系统模拟Http的get或post请求

Http请求指的是客户端向服务器的请求消息，Http请求主要分为get或post两种，在Linux系统下可以用curl和wget命令来模拟Http的请求。下面就来介绍一下Linux系统如何模拟Http的get或post请求。

#### 一、get请求：
	
	1、使用curl命令：
	
	curl “http://www.baidu.com” 如果这里的URL指向的是一个文件或者一幅图都可以直接下载到本地
	curl -i “http://www.baidu.com” 显示全部信息
	curl -l “http://www.baidu.com” 只显示头部信息
	curl -v “http://www.baidu.com” 显示get请求全过程解析

	2、使用wget命令：
	wget “http://www.baidu.com”也可以

#### 二、post请求

	1、使用curl命令（通过-d参数，把访问参数放在里面）：
	
	curl -d “param1=value1¶m2=value2” “http://www.baidu.com”

	2、使用wget命令：（--post-data参数来实现）
	wget --post-data ‘user=foo&password=bar’ http://www.baidu.com

以上就是Linux模拟Http的get或post请求的方法了，这样一来Linux系统也能向远程服务器发送消息了。