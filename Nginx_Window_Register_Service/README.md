### Window平台下Nginx注册为后台服务  ###
Nginx 不是为 Windows 而写。Nginx 是用在软件的工作环境中的。但软件开发环境一般都是 Windows，有时调试的需要也要装 Nginx，但 Nginx 并没给 Windows 提供服务支持。如何把 Nginx 创建为 Windows 的一个服务呢？

把 Nginx 创建为 Windows 的一个服务，比较流行的一个做法就是用微软提供的 instsrv/servany。本文没有使用这种做法，并说明理由。

#### 引言 ####

Nginx 是一个 web 服务器。它类似于 Lighttpd，作为轻量级的 web server，可以替代重量级的 Apache/IIS。Nginx 专为性能优化而开发，是一个快速且能经受高负载考验的 web server。它来自于 Linux 世界但同样可以运行在 Windows 上面(由本地语言构建)。唯一的问题就是它不支持 Windows Services。

尽管 Nginx 以快速和提供高性能而具有很大的声誉，但并非是在 Windows 平台上。访问官方网址 [http://nginx.org/en/docs/windows.html](http://nginx.org/en/docs/windows.html)，你会发现，对于 Windows 平台的支持被认为是测试版，根据 Nginx 的实现来看它并不提供(和 Linux 平台)相同的性能水平。

也许你都没有意识到，WordPress 就是一个 Nginx 的用户，使用它提供大量的静态内容服务，并负载平衡请求到其他服务器。如果你想了解更多关于 Nginx 的内容请看本文底部的链接。

#### Nginx 对比 Windows 服务 ####
Nginx 是绿色免安装的。这里我不去介绍它的管理配置，官方已经提供了一个很棒的 wiki，上面有丰富的服务器相关信息(参考文后链接)。可以使用默认的配置，它会使用 Nginx 目录下的 html 文件夹服务于端口 80。

只需简单地执行 nginx.exe 即可启动 Nginx。但你想要停止它的时候问题来了，你需要执行以下命令：

	nginx.exe -s stop  	

虽然这很简单，但是如果它能够像 apache 或 IIS 那样作为一个服务工作的话会更漂亮。那样的话，我们就可以设置机器启动时 Nginx 自动启动，还可以方便地启动、停止或者重启服务，设置恢复选项、依赖的服务，等等。

为什么不使用 instsrv/servany、FireDaemon 或者其他办法呢?

已经有介绍如何通过 FireDaemon 使用 Nginx，但它有一个很重要的问题。Nginx 启动以后，它会创建一个次级进程。所以会有两个 nginx.exe 在运行。对于这个官方可能有一个很合理的理由，但你最好到 Nginx 论坛去问为什么:-)

通过 instsrv/srvany(微软官方创建服务的方法) 或者 FireDaemon 的方式(来创建 Nginx 为服务)，只是启动进程，当你想要停止它时，将关闭这个进程。但这些方式都无法关闭多余的那个 nginx.exe 进程。所以每次你停止/启动/重启服务都会产生一个多余的 nginx.exe 进程。不怎么好！

把 Nginx 创建为 Windows 的一个服务(一个较好的做法)

 多亏了一个叫做 ["Windows Service Wrapper"](http://projectkenai.com/projects/winsw) 的小项目，我们有了一个办法来恰当地启动和停止 Nginx。首先从[https://github.com/scalad/Note/blob/master/Nginx_Window_Register_Service/winsw-1.8-bin.zip](https://github.com/scalad/Note/blob/master/Nginx_Window_Register_Service/winsw-1.8-bin.zip)下载winsw工具包

得到该程序后，将其放在 Nginx 安装目录下，并重命名为 myapp.exe。

然后是告诉 WinSw 我们想要它做什么。这将使用一个 XML 配置文件，我们将在文件中指出 Nginx 需要一个 shutdown 命令。

(在 Nginx 安装目录下)新建一个名为 myapp.xml 的文件，编辑其内容如下：

```xml
<service>  
 <id>nginx</id>  
 <name>nginx</name>  
 <description>nginx</description>  
 <executable>c:\nginx\nginx.exe</executable>  
 <logpath>c:\nginx\</logpath>  
 <logmode>roll</logmode>  
 <depend></depend>  
 <startargument>-p c:\nginx</startargument>  
 <stopargument>-p c:\nginx -s stop</stopargument>  
</service>  
```

很明显，你应该稍微更改文件，这取决于你自己的文件路径。对于有更多技术需求的朋友，你也可以在该文件中设置 Nginx 依赖的服务。最后，我们要安装服务了。只需要简单地执行以下语句，你将在你的服务列表
里找到 "Nginx" 服务：

	c:\nginx\myapp.exe install

就这些！

结束语

根据我的经验，到目前为止这种做法的效果很完美。你得到了 Windows 服务的支持，而且在服务重启时没有遗留孤立的 "nginx.exe"。两全其美。
如果 Nginx 自己可以做到这样的话会更好，但 Nginx 的作者当下正在专注于其他更重要的开发。我敢肯定还有其他人有足够的编程知识来贡献这块所需的代码，所以，如果你是这样的一个人，请尽力来帮助大家。  