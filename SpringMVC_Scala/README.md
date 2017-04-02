### Scala和Spring集成(Scala和Java完成共同编译)

Scala看起来像是一种纯粹的面向对象编程语言，而又无缝地结合了命令式和函数式的编程风格。前日，Groovy创始人撰博文称Scala将取代Java。他说，如果他在2003年看过《Programming Scala》的话，那可能就不会有Groovy了。

Scala相对Java来说简洁，Scala结合了面向对象编程与函数编程思想，使用一种能够完全兼容Java、可以运行在Java虚拟机上的、简洁的语法。对于函数编程风格的支持，尤其是对于Lambda表达式的支持(Java8 也对Lambda表达式支持)，能够有助于减少必须要编写的逻辑无关固定代码，也许让它可以更简单的关注要面对的任务本身。

目前 Scala 的影响力也在缓慢扩大, 比如 Scala 社区中的明星 Spark 的流行也在慢慢拉动 Scala 的流行, 如同 rails 之于 ruby。

为了满足Java程序员在Java平台上开发Scala程序，我们可以在项目中混合使用Scala和Java，我们使用Gradle构建项目框架

首先你的环境下必须有这些东西

* Scala2.1
* Gradle3.1
* JDK1.7+

好了，我们开始了