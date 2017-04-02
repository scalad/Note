### Shiro Session
session管理可以说是Shiro的一大卖点

Shiro可以为任何应用(从简单的命令行程序还是手机应用再到大型企业应用)提供会话解决方案。

在Shiro出现之前，如果我们想让你的应用支持session，我们通常会依赖web容器或者使用EJB的Session Bean。

Shiro对session的支持更加易用，而且他可以在任何应用、任何容器中使用。

即便我们使用Servlet或者EJB也并不代表我们必须使用容器的session，Shiro提供的一些特性足以让我们用Shiro session替代他们。

* 基于POJO

* 易定制session持久化

* 容器无关的session集群

* 支持多种客户端访问

* 会话事件监听

* 对失效session的延长

* 对Web的透明支持

* 支持SSO
 
使用Shiro session时，无论是在JavaSE还是web，方法都是一样的。

```Java
public static void main(String[] args) {
    Factory<SecurityManager> factory = new IniSecurityManagerFactory("classpath:shiro/shiro.ini");

    SecurityUtils.setSecurityManager(factory.getInstance());
    Subject currentUser = SecurityUtils.getSubject();
    UsernamePasswordToken token = new UsernamePasswordToken("king","t;stmdtkg");
    currentUser.login(token);

    Session session = currentUser.getSession();
    System.out.println(session.getHost());
    System.out.println(session.getId());

    System.out.println(session.getStartTimestamp());
    System.out.println(session.getLastAccessTime());

    session.touch();
    User u = new User(); 
    session.setAttribute(u, "King.");
    Iterator<Object> keyItr = session.getAttributeKeys().iterator();
    while(keyItr.hasNext()){
        System.out.println(session.getAttribute(keyItr.next()));
    }
}
```

无论是什么环境，只需要调用Subject的getSession()即可。
另外Subject还提供了一个...

	Session getSession(boolean create);

即，当前Subject的session不存在时是否创建并返回新的session。

即，当前Subject的session不存在时是否创建并返回新的session。


以DelegatingSubject为例：
(注意!从Shiro 1.2开始多了一个isSessionCreationEnabled属性，其默认值为true。)

```Java
public Session getSession() {
    return getSession(true);
}

public Session getSession(boolean create) {
    if (log.isTraceEnabled()) {
        log.trace("attempting to get session; create = " + create +
                "; session is null = " + (this.session == null) +
                "; session has id = " + (this.session != null && session.getId() != null));
    }

    if (this.session == null && create) {

        //added in 1.2:
        if (!isSessionCreationEnabled()) {
            String msg = "Session creation has been disabled for the current subject.  This exception indicates " +
                    "that there is either a programming error (using a session when it should never be " +
                    "used) or that Shiro's configuration needs to be adjusted to allow Sessions to be created " +
                    "for the current Subject.  See the " + DisabledSessionException.class.getName() + " JavaDoc " +
                    "for more.";
            throw new DisabledSessionException(msg);
        }

        log.trace("Starting session for host {}", getHost());
        SessionContext sessionContext = createSessionContext();
        Session session = this.securityManager.start(sessionContext);
        this.session = decorate(session);
    }
    return this.session;
}
```

### SessionManager

正如其名，sessionManager用于为应用中的Subject管理session，比如创建、删除、失效或者验证等。
和Shiro中的其他核心组件一样，他由SecurityManager维护。
> (注意：public interface SecurityManager extends Authenticator, Authorizer, SessionManager)。

```Java
public interface SessionManager {
    Session start(SessionContext context);
    Session getSession(SessionKey key) throws SessionException;
}
```

Shiro为SessionManager提供了3个实现类(顺便也整理一下与SecurityManager实现类的关系)。

![](https://github.com/scalad/Note/blob/master/Java_Shiro_Session/image/012002107171657.jpg)

* DefaultSessionManager

* DefaultWebSessionManager

* ServletContainerSessionManager

其中ServletContainerSessionManager只适用于servlet容器中，如果需要支持多种客户端访问，则应该使用DefaultWebSessionManager。

默认情况下，sessionManager的实现类的超时设为30分钟。

见AbstractSessionManager：

	public static final long DEFAULT_GLOBAL_SESSION_TIMEOUT = 30 * MILLIS_PER_MINUTE;
	private long globalSessionTimeout = DEFAULT_GLOBAL_SESSION_TIMEOUT;


当然，我们也可以直接设置AbstractSessionManager的globalSessionTimeout。

比如在.ini中：

	securityManager.sessionManager.globalSessionTimeout = 3600000

注意！如果使用的SessionManager是ServletContainerSessionManager(没有继AbstractSessionManager)，超时设置则依赖于Servlet容器的设置。
见： [https://issues.apache.org/jira/browse/SHIRO-240](https://issues.apache.org/jira/browse/SHIRO-240) 

session过期的验证方法可以参考SimpleSession：

```Java
protected boolean isTimedOut() {

    if (isExpired()) {
        return true;
    }

    long timeout = getTimeout();

    if (timeout >= 0l) {

        Date lastAccessTime = getLastAccessTime();

        if (lastAccessTime == null) {
            String msg = "session.lastAccessTime for session with id [" +
                    getId() + "] is null.  This value must be set at " +
                    "least once, preferably at least upon instantiation.  Please check the " +
                    getClass().getName() + " implementation and ensure " +
                    "this value will be set (perhaps in the constructor?)";
            throw new IllegalStateException(msg);
        }

        // Calculate at what time a session would have been last accessed
        // for it to be expired at this point.  In other words, subtract
        // from the current time the amount of time that a session can
        // be inactive before expiring.  If the session was last accessed
        // before this time, it is expired.
        long expireTimeMillis = System.currentTimeMillis() - timeout;
        Date expireTime = new Date(expireTimeMillis);
        return lastAccessTime.before(expireTime);
    } else {
        if (log.isTraceEnabled()) {
            log.trace("No timeout for session with id [" + getId() +
                    "].  Session is not considered expired.");
        }
    }

    return false;
}
```

试着从SecurityUtils.getSubject()一步步detect，感受一下session是如何设置到subject中的。
判断线程context中是否存在Subject后，若不存在，我们使用Subject的内部类Builder进行buildSubject();

```Java
public static Subject getSubject() {
    Subject subject = ThreadContext.getSubject();
    if (subject == null) {
        subject = (new Subject.Builder()).buildSubject();
        ThreadContext.bind(subject);
    }
    return subject;
}
```

buildSubject()将建立Subject的工作委托给securityManager.createSubject(subjectContext)
createSubject会调用resolveSession处理session。

```Java
protected SubjectContext resolveSession(SubjectContext context) {
    if (context.resolveSession() != null) {
        log.debug("Context already contains a session.  Returning.");
        return context;
    }
    try {
        //Context couldn't resolve it directly, let's see if we can since we have direct access to 
        //the session manager:
        Session session = resolveContextSession(context);
        if (session != null) {
            context.setSession(session);
        }
    } catch (InvalidSessionException e) {
        log.debug("Resolved SubjectContext context session is invalid.  Ignoring and creating an anonymous " +
                "(session-less) Subject instance.", e);
    }
    return context;
}
```

resolveSession(subjectContext)，首先尝试从context(MapContext)中获取session，如果无法直接获取则改为获取subject，再调用其getSession(false)。

如果仍不存在则调用resolveContextSession(subjectContext)，试着从MapContext中获取sessionId。

根据sessionId实例化一个SessionKey对象，并通过SessionKey实例获取session。

getSession(key)的任务直接交给sessionManager来执行。

```Java
public Session getSession(SessionKey key) throws SessionException {
    return this.sessionManager.getSession(key);
}
```

sessionManager.getSession(key)方法在AbstractNativeSessionManager中定义，该方法调用lookupSession(key)，

lookupSession调用doGetSession(key)，doGetSession(key)是个protected abstract，实现由子类AbstractValidatingSessionManager提供。

doGetSession调用retrieveSession(key)，该方法尝试通过sessionDAO获得session信息。

最后，判断session是否为空后对其进行验证(参考SimpleSession.validate())。

```Java
protected final Session doGetSession(final SessionKey key) throws InvalidSessionException {
    enableSessionValidationIfNecessary();

    log.trace("Attempting to retrieve session with key {}", key);

    Session s = retrieveSession(key);
    if (s != null) {
        validate(s, key);
    }
    return s;
}
```

Session Listener

我们可以通过SessionListener接口或者SessionListenerAdapter来进行session监听，在session创建、停止、过期时按需进行操作。

```Java
public interface SessionListener {

    void onStart(Session session);

    void onStop(Session session);

    void onExpiration(Session session);
}
```

我只需要定义一个Listener并将它注入到sessionManager中。

```Java
package pac.testcase.shiro.listener;

import org.apache.shiro.session.Session;
import org.apache.shiro.session.SessionListener;

public class MySessionListener implements SessionListener {

    public void onStart(Session session) {
        System.out.println(session.getId()+" start...");
    }

    public void onStop(Session session) {
        System.out.println(session.getId()+" stop...");
    }

    public void onExpiration(Session session) {
        System.out.println(session.getId()+" expired...");
    }

}
```

	[main]
	realm0=pac.testcase.shiro.realm.MyRealm0
	realm1=pac.testcase.shiro.realm.MyRealm1
	
	
	authcStrategy = org.apache.shiro.authc.pam.AllSuccessfulStrategy
	sessionManager = org.apache.shiro.web.session.mgt.DefaultWebSessionManager
	#sessionManager = org.apache.shiro.web.session.mgt.ServletContainerSessionManager
	
	sessionListener = pac.testcase.shiro.listener.MySessionListener
	securityManager.realms=$realm1
	securityManager.authenticator.authenticationStrategy = $authcStrategy
	securityManager.sessionManager=$sessionManager
	#sessionManager.sessionListeners =$sessionListener  
	securityManager.sessionManager.sessionListeners=$sessionListener

###SessionDAO

SessionManager将session CRUD的工作委托给SessionDAO。

我们可以用特定的数据源API实现SessionDAO，以将session存储于任何一种数据源中。

```Java
public interface SessionDAO {

    Serializable create(Session session);

    Session readSession(Serializable sessionId) throws UnknownSessionException;

    void update(Session session) throws UnknownSessionException;

    void delete(Session session);

    Collection<Session> getActiveSessions();
}
```

![](https://github.com/scalad/Note/blob/master/Java_Shiro_Session/image/012002553111967.jpg)

* AbstractSessionDAO：在create和read时对session做验证，保证session可用，并提供了sessionId的生成方法。

* CachingSessionDAO：为session存储提供透明的缓存支持，使用CacheManager维护缓存。

* EnterpriseCacheSessionDAO：通过匿名内部类重写了AbstractCacheManager的createCache，返回MapCache对象。

* MemorySessionDAO：基于内存的实现，所有会话放在内存中。

![](https://github.com/scalad/Note/blob/master/Java_Shiro_Session/image/012003031238347.jpg)

默认使用MemorySessionDAO(注意！DefaultWebSessionManager extends DefaultSessionManager)

 

当然，我们也可以试着使用缓存。
Shiro没有默认启用EHCache，但是为了保证session不会在运行时莫名其妙地丢失，建议启用EHCache优化session管理。
启用EHCache为session持久化服务非常简单，首先我们需要添加一个denpendency。

```Xml
<dependency>
    <groupId>org.apache.shiro</groupId>
    <artifactId>shiro-ehcache</artifactId>
    <version>${shiro.version}</version>
</dependency>
```

接着只需要配置一下，以.ini配置为例：

	[main]
	realm0=pac.testcase.shiro.realm.MyRealm0
	realm1=pac.testcase.shiro.realm.MyRealm1
	
	authcStrategy = org.apache.shiro.authc.pam.AllSuccessfulStrategy
	sessionManager = org.apache.shiro.web.session.mgt.DefaultWebSessionManager
	cacheManager=org.apache.shiro.cache.ehcache.EhCacheManager
	sessionDAO=org.apache.shiro.session.mgt.eis.EnterpriseCacheSessionDAO
	#sessionManager = org.apache.shiro.web.session.mgt.ServletContainerSessionManager
	
	sessionListener = pac.testcase.shiro.listener.MySessionListener
	securityManager.realms=$realm1
	securityManager.authenticator.authenticationStrategy = $authcStrategy
	securityManager.sessionManager=$sessionManager
	sessionManager.sessionListeners =$sessionListener
	sessionDAO.cacheManager=$cacheManager  
	securityManager.sessionManager.sessionDAO=$sessionDAO
	securityManager.sessionManager.sessionListeners=$sessionListener

此处主要是cacheManager的定义和引用。

另外，此处使用的sessionDAO为EnterpriseCacheSessionDAO。

前面说过EnterpriseCacheSessionDAO使用的CacheManager是基于MapCache的。

其实这样设置并不会影响，因为EnterpriseCacheSessionDAO继承CachingSessionDAO，CachingSessionDAO实现CacheManagerAware。

注意！只有在使用SessionManager的实现类时才有sessionDAO属性。

(事实上他们把sessionDAO定义在DefaultSessionManager中了，但似乎有将sessionDAO放到AbstractValidatingSessionManager的打算。)

如果你在web应用中配置Shiro，启动后你会惊讶地发现securityManger的sessionManager属性居然是ServletContainerSessionManager。

看一下上面的层次图发现ServletContainerSessionManager和DefaultSessionManager没有关系。
也就是说ServletContainerSessionManager不支持SessionDAO(cacheManger属性定义在CachingSessionDAO)。

此时需要显示指定sessionManager为DefaultWebSessionManager。

关于EhCache的配置，默认情况下EhCacheManager使用指定的配置文件，即：

	private String cacheManagerConfigFile = "classpath:org/apache/shiro/cache/ehcache/ehcache.xml";

来看一下他的配置：

```Xml
<ehcache>
    <diskStore path="java.io.tmpdir/shiro-ehcache"/>
    <defaultCache
            maxElementsInMemory="10000"
            eternal="false"
            timeToIdleSeconds="120"
            timeToLiveSeconds="120"
            overflowToDisk="false"
            diskPersistent="false"
            diskExpiryThreadIntervalSeconds="120"
            />

    <cache name="shiro-activeSessionCache"
           maxElementsInMemory="10000"
           overflowToDisk="true"
           eternal="true"
           timeToLiveSeconds="0"
           timeToIdleSeconds="0"
           diskPersistent="true"
           diskExpiryThreadIntervalSeconds="600"/>

    <cache name="org.apache.shiro.realm.text.PropertiesRealm-0-accounts"
           maxElementsInMemory="1000"
           eternal="true"
           overflowToDisk="true"/>

</ehcache>
```

如果打算改变该原有设置，其中有两个属性需要特别注意：

* overflowToDisk="true"：保证session不会丢失。

* eternal="true"：保证session缓存不会被自动失效，将其设为false可能会和session validation的逻辑不符。

另外，name默认使用"shiro-activeSessionCache"

public static final String ACTIVESESSIONCACHE_NAME = "shiro-activeSessionCache";

如果打算使用其他名字，只要在CachingSessionDAO或其子类设置activeSessionsCacheName即可。

当创建一个新的session时，SessionDAO的实现类使用SessionIdGenerator来为session生成ID。
默认使用的SessionIdGenerator是JavaUuidSessionIdGenerator，其实现为：

```Java
public Serializable generateId(Session session) {
    return UUID.randomUUID().toString();
}
```

当然，我们也可以自己定制实现SessionIdGenerator。 

### Session Validation & Scheduling

比如说用户在浏览器上使用web应用时session被创建并缓存什么的都没有什么问题，只是用户退出的时候可以直接关掉浏览器、关掉电源、停电或者其他天灾什么的。

然后session的状态就不得而知了(it is orphaned)。
为了防止垃圾被一点点堆积起来，我们需要周期性地检查session并在必要时删除session。
于是我们有SessionValidationScheduler：

```Java
public interface SessionValidationScheduler {

    boolean isEnabled();
    void enableSessionValidation();
    void disableSessionValidation();

}
```

Shiro只提供了一个实现，ExecutorServiceSessionValidationScheduler。 默认情况下，验证周期为60分钟。

当然，我们也可以通过修改他的interval属性改变验证周期(单位为毫秒)，比如这样：

	sessionValidationScheduler = org.apache.shiro.session.mgt.ExecutorServiceSessionValidationScheduler
	sessionValidationScheduler.interval = 3600000
	securityManager.sessionManager.sessionValidationScheduler = $sessionValidationScheduler

如果打算禁用按周期验证session(比如我们在Shiro外做了一些工作)，则可以设置

	securityManager.sessionManager.sessionValidationSchedulerEnabled = false

如果不打算删除失效的session(比如我们要做点统计之类的)，则可以设置

	securityManager.sessionManager.deleteInvalidSessions = false