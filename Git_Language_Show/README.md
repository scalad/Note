### 修改Github上项目语言显示问题

#### 问题
最近将自己写的博客放到github上了。由于使用了富文本编辑器、jQuery、Bootstrap等第三方插件，导致js、css等代码远远超过你自己写的代码

于是也就成这样了

![](https://github.com/scalad/Note/blob/master/Git_Language_Show/image/vQz6FjB.png)

而且这里也显示JavaScript

![](https://github.com/scalad/Note/blob/master/Git_Language_Show/image/EBFZJrr.png)

这样的情况很不能忍，尤其对于强迫症来说。而且github也没有bitbucket项目语言的设置

搜索了一下发现github是使用 Linguist 来detect所使用的语言。 Linguist 是什么鬼我也不了解，大致就是通过统计哪种语言代码数量最多的作为当前项目主语言。这样很不公平有木有，像Scala这种支持函数式编程而且语法简洁的语言，代码量完全拼不过其他语言

#### 解决

解决起来也简单，有2种方法

#### 1、使用外链接

将项目中的静态文件如jQuery、Bootstrap等放到别处用连接导入即可

#### 2、使用 .gitattributes 配置文件

具体就是在项目根目录添加文件名为.gitattributes的文本文件，写入

	*.js linguist-language=Scala
	*.css linguist-language=Scala
	*.html linguist-language=Scala

意思就是将.js、css、html当作Scala语言来统计

另外，说一下，在windows系统中并不好直接创建名为 .gitattributes 的文件，会提示:

![](https://github.com/scalad/Note/blob/master/Git_Language_Show/image/u2UVna.png)

那么只需要用命令行创建就行了,或者使用VI也是可以的：

touch .gitattributes