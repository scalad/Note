#### Spring Security 3.1 中功能强大的加密工具 PasswordEncoder
去年发生的密码泄漏事件，我们也对密码加密做了重新研究。 
  
在筛选加密方法的过程中，发现了Spring Security 3.1.0版本中提供了新的PasswordEncoder，它的加密方法非常给力！虽然ns同学曾经说过“你的网站看起来很安全, 只是因为人家没精力或者没兴趣搞你...”，但是找到一个好的加密方法，无疑还是会有很大帮助的，至少会延迟破解的时间。  

说到PasswordEncoder，使用过Spring Security的人应该不会陌生。在3.1.0版本之前，位于org.springframework.security.authentication.encoding包中，辅以一系列的实现类，每个实现类使用了不同的加密方法，有Md4PasswordEncoder、Md5PasswordEncoder、ShaPasswordEncoder、PlaintextPasswordEncoder等。加密方法采用hash+salt方式。 

3.0.0版本中PasswordEncoder继承关系

![](https://github.com/silence940109/Java/blob/master/image/Security3.0.jpg "Spring Security 3.0")

在3.1.0及之后的版本，增加了crypto包/模块，提供了更加强大的对称加密，生成key，以及密码加密功能。这个模块是作为core的一部分存在的，但是对其它任何Spring Security或Spring代码没有依赖，也可以单独作为一个包存在！ 

3.1.0版本中新的PasswordEncoder继承关系 

![](https://github.com/silence940109/Java/blob/master/image/Security3.1.jpg "Spring Security 3.1")

在3.1.0以前的版本中，PasswordEncoder位于 
org.springframeword.security.authentication.encoding包中，接口定义如下： 

	public interface PasswordEncoder{  
	   String encodePassword(String rawPass,Object salt);  
	   Boolean isPasswordValid(String encPass,String rawPass,Object salt);  
	} 

接口中定义了两个方法，encodePassword()方法是对原始密码进行加密，采用hash+salt方式，在方法中得提供盐值(salt)。 isPasswordValid方法是用来验证密码是否正确的，得提供三个参数，原始密码，加密后的密码以及盐值(salt)。 

在具体加密过程中，可以动态选择各种实现类(其实就是各种加密算法)，配合salt来进行加密和验证工作。 

缺点就是每次加密和解密都得提供盐值，那么有两种方式，一是将盐值和密码存在一起，一是使用固定的盐值。盐值和密码存在一起也不是一个好的方案，如果数据库被盗，有了密码和盐值，暴力破解会容易一些；而固定的盐值也不是一个很好的方案。

Spring小组意识到了之前的PasswordEncoder的缺点，于是在3.1.0中推出了新的、更给力的PasswordEncoder，为了向前兼容，不能废掉PasswordEncoder，于是在另一个包中重新定义了一个PasswordEncoder接口。不过个人愚见，应该在org.springframeword.security.authentication.encoding.PasswordEncoder接口上增加@Deprecated 

而在Spring-Security 3.1.0 版本之后，Spring-security-crypto模块中的password包提供了更给力的加密密码的支持，这个包中也有PasswordEncoder接口，接口定义如下。

	Public interface PasswordEncoder{  
	  String encode(String rawPassword);  
	  Boolean matches(String rawPassword,String encodedPassword);  
	}  

定义了两个方法，encode方法是对方法加密，而match方法是用来验证密码和加密后密码是否一致的，如果一致则返回true。和authentication.encoding包中的PasswordEncoder接口相比，简化了许多。 

位于org.springframeword.security.crypto.password包中的 
StandardPasswordEncoder类，是PasswordEncoder接口的(唯一)一个实现类，是本文所述加密方法的核心。它采用SHA-256算法，迭代1024次，使用一个密钥(site-wide secret)以及8位随机盐对原密码进行加密。 随机盐确保相同的密码使用多次时，产生的哈希都不同； 密钥应该与密码区别开来存放，加密时使用一个密钥即可；对hash算法迭代执行1024次增强了安全性，使暴力破解变得更困难些。

和上一个版本的PasswordEncoder比较，好处显而易见：盐值不用用户提供，每次随机生成；多重加密————迭代SHA算法+密钥+随机盐来对密码加密，大大增加密码破解难度。

简单封装一下：

如果你使用的maven，可以加入依赖

		<!-- https://mvnrepository.com/artifact/org.springframework.security/spring-security-core -->
		<dependency>
			<groupId>org.springframework.security</groupId>
			<artifactId>spring-security-core</artifactId>
			<version>4.1.3.RELEASE</version>
		</dependency>

#		
	import org.springframework.security.crypto.password.PasswordEncoder;
	import org.springframework.security.crypto.password.StandardPasswordEncoder;
	
	/**
	 * Spring Security
	 *
	 */
	public class Security {
	// 从配置文件中获得
	private static final String SITE_WIDE_SECRET = "beibei";
	private static final PasswordEncoder encoder = new StandardPasswordEncoder(SITE_WIDE_SECRET);

	public static String encrypt(String rawPassword) {
		return encoder.encode(rawPassword);
	}

	public static boolean match(String rawPassword, String password) {
		return encoder.matches(rawPassword, password);
	}

	public static void main(String[] args) {
		System.out.println(Security.encrypt("test"));
		System.out.println(Security.encrypt("test"));
		System.out.println(Security.encrypt("test"));
		System.out.println(Security.match("test", Security.encrypt("test")));
		}
    }
#
从配置文件中获得密钥（SITE_WIDE_SECRET），使用static的PasswordEncoder，保证只用一个实例来加密与 
匹配。 

加密后得到的密码是80位，霸气十足。

	f40f9cb79ea2a9827599b49285ee6f83086540c762f17cba283253d53689d912b3b4decae83b0325
	fae4f9a8101d8f425ff075d4a67bfe75ef2677d65341d16673d2e9e55283a3a863f630328d173b32
	324205c207a27db03cbb2c44d00880ecf7850842b49ce2f6d8d0a1e7259f4b3e482b67b6b38faed7
	true

	
