### 弱密码摄像头批量入侵

##### 弱密码摄像头批量入侵，端口37777,8000，默认密码账号admin
这个一个非常简单易懂，没有什么太多技术含量的，默认弱密码摄像头入侵。

首先，使用Scanport 批量扫描你所选IP的此类型号摄像头的端口，37777,8000，也可以指定IP或者扫自己周围IP的，这样比较快，如果说范围太大可能会很慢。

![](https://github.com/silence940109/Java/blob/master/Security_Camera_Invade/image/2016063013591821.png)

(海康的服务控制端口是8000，大华的是37777，经过测试确实这两款还是存在不少弱密码没有修改，虽然很多地方都爆料过，但是现在大部分地区对于网络安全还是缺乏安全意识。)

第二步，打开工具PSS4.6参数生成器，把全部已经搜索到的IP，直接复制到PSS4.6参数生成器当中，配置好PSS的路径和密码，点击输出即可。

![](https://github.com/silence940109/Java/blob/master/Security_Camera_Invade/image/2016063014025861.png)

![](https://github.com/silence940109/Java/blob/master/Security_Camera_Invade/image/2016063014031133.png)

接下来就可以登录PSS，发现输出的密码已经填好，直接登录即可。

![](https://github.com/silence940109/Java/blob/master/Security_Camera_Invade/image/2016063014062927.jpg)

然后点登录，就进去了，但是，现在发现有很多都是查看不了的，可能是很多原因因素导致的，还需要自己更深入的去了解，想办法去解决，如果只是随便看看，当中肯定是有一两个是可以查看的。

![](https://github.com/silence940109/Java/blob/master/Security_Camera_Invade/image/2016063014092681.jpg)

工具下载地址：链接：http://pan.baidu.com/s/1pKOfFgn 密码：xwn8