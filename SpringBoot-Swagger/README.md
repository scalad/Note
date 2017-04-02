### SpringBoot-Scala-Swagger

swagger用于定义API文档。

好处：

* 前后端分离开发
* API文档非常明确
* 测试的时候不需要再使用URL输入浏览器的方式来访问Controller
* 传统的输入URL的测试方式对于post请求的传参比较麻烦（当然，可以使用postman这样的浏览器插件）
* spring-boot与swagger的集成简单的一逼

Maven项目

```Xml
  <dependency>
     <groupId>io.springfox</groupId>
     <artifactId>springfox-swagger2</artifactId>
     <version>2.2.2</version>
  </dependency>
  <dependency>
     <groupId>io.springfox</groupId>
     <artifactId>springfox-swagger-ui</artifactId>
     <version>2.2.2</version>
  </dependency>
```

Gradle项目

	compile ("io.springfox:springfox-swagger2:2.2.2")
    compile ("io.springfox:springfox-swagger-ui:2.2.2")

SpringBoot启动应用时加上@EnableSwagger2注解

```Java

@Configuration
@EnableAutoConfiguration
@ComponentScan
@SpringBootApplication
//启动swagger注解
@EnableSwagger2
class Config

object StartSpringBootApplication extends App {

  SpringApplication.run(classOf[Config])
    
}

```

在controller层加入swagger api注解

Scala模式

```Java

@ComponentScan
@Controller
@ResponseBody
@Api(value = "用户相关操作")
class UserController @Autowired()(private val userService : UserService){
    
    @ApiOperation("查询用户信息")
    @ApiImplicitParams(Array(new ApiImplicitParam(paramType="header",name="id",dataType="Integer",required=true,value="用户的编号",defaultValue="1")))
    @ApiResponses(Array(new ApiResponse(code=400,message="请求参数没填好"),new ApiResponse(code=404,message="请求路径没有或页面跳转路径不对")))
  	@RequestMapping(value = Array("/find/{id}"), method = Array(RequestMethod.GET))
    def find(@PathVariable(value = "id") id: Long) : User = {
        userService.find(id)
    }
    
}
```

Java模式

```Java
@RestController
@RequestMapping("/user")
@Api("userController相关api")
public class UserController {

    @Autowired
    private UserService userService;
    
    @ApiOperation("获取用户信息")
    @ApiImplicitParams({
        @ApiImplicitParam(paramType="header",name="username",dataType="String",required=true,value="用户的姓名",defaultValue="zhaojigang"),
        @ApiImplicitParam(paramType="query",name="password",dataType="String",required=true,value="用户的密码",defaultValue="wangna")
    })
    @ApiResponses({
        @ApiResponse(code=400,message="请求参数没填好"),
        @ApiResponse(code=404,message="请求路径没有或页面跳转路径不对")
    })
    @RequestMapping(value="/getUser",method=RequestMethod.GET)
    public User getUser(@RequestHeader("username") String username, @RequestParam("password") String password) {
        return userService.getUser(username,password);
    }
}
```

说明：

* @Api：用在类上，说明该类的作用
* @ApiOperation：用在方法上，说明方法的作用
* @ApiImplicitParams：用在方法上包含一组参数说明
* @ApiImplicitParam：用在@ApiImplicitParams注解中，指定一个请求参数的各个方面

	- paramType：参数放在哪个地方
	- header-->请求参数的获取：@RequestHeader
	- query-->请求参数的获取：@RequestParam
	- path（用于restful接口）-->请求参数的获取：@PathVariable
	- body（不常用）
	- form（不常用）
	- name：参数名
	- dataType：参数类型
	- required：参数是否必须传
	- value：参数的意思
	- defaultValue：参数的默认值

* @ApiResponses：用于表示一组响应
* @ApiResponse：用在@ApiResponses中，一般用于表达一个错误的响应信息
	
	- code：数字，例如400
	- message：信息，例如"请求参数没填好"
	- response：抛出异常的类

* @ApiModel：描述一个Model的信息（这种一般用在post创建的时候，使用@RequestBody这样的场景，请求参数无法使用@ApiImplicitParam注解进行描述的时候）
* @ApiModelProperty：描述一个model的属性

以上这些就是最常用的几个注解了。

[项目地址](https://github.com/silence940109/WebSocket)